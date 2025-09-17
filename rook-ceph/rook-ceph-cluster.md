# Installation

Ref: https://rook.io/docs/rook/latest/Getting-Started/quickstart/



# Requirement: 

1. kubernets cluster with at least 3 nodes (kubernetes masters can be used as workers after removing taints)
2. A disk  attached to each nodes to be used as OSD. 
3. 2 NIC one for Public Network and 2nd for Cluster Network
4. Public network will be used to access ceph cluster services.
5. Cluster network will be used by ceph cluster for internal operations like, recovery, balancing of data etc.
6. If re-installing ceph , please make sure to clear directory /var/lib/rook . otherwise it will not allow mon to joing cluster.


# Clone rook-ceph git repository

```
git clone --single-branch --branch master https://github.com/rook/rook.git

cd rook/deploy/example
```

# Install rook-ceph operators

```
kubectl create -f crds.yaml -f common.yaml -f csi-operator.yaml -f operator.yaml

# verify the rook-ceph-operator is in the `Running` state before proceeding

kubectl -n rook-ceph get pod
```

There are few setting that need to be verified before we install operator 

ROOK_LOG_LEVEL: "INFO"
ROOK_ENABLE_DISCOVERY_DAEMON: "false"
CSI_CLUSTER_NAME: "my-prod-cluster"



# Installing Ceph cluster

Check Current parameters:

```
cd ~rook/deploy/example
vi cluster.yaml

```

```
spec:
  cephVersion: 
    image: quay.io/ceph/ceph:v19.2.0
  dataDirHostPath: /var/lib/rook     #The path on the host where configuration files will be persisted. Important: if you reinstall the cluster, make sure you delete this directory from each host or else the mons will fail to start on the new cluster.

  network:
    provider: host    #Ref: https://rook.io/docs/rook/latest/CRDs/Cluster/network-providers/#specifying-ceph-network-selections
    addressRanges:
        public:
          - "192.168.122.0/24"
        cluster:
          - "10.10.10.0/24"
  storage: # cluster level storage configuration and selection
    useAllNodes: true
    useAllDevices: true   
    #deviceFilter: "^sd."     
    config:
      # databaseSizeMB: "1024" # uncomment if the disks are smaller than 100 GB
      # osdsPerDevice: "1" # this value can be overridden at the node or device level
```

# Get Cluster status:
```
kubectl -n rook-ceph get pod
kubectl get cephclusters.ceph.rook.io -n rook-ceph
```

# Install kubectl Plugin

Download and install krew on Linux

Ref: https://krew.sigs.k8s.io/docs/user-guide/setup/install/  OR Install using kubespary whie created cluster or re-run cluster.yaml with tag krew

Install Rook plugin

```
kubectl krew install rook-ceph
```
```
kubectl krew list
```
Test krew plugin 

Ref: https://github.com/rook/kubectl-rook-ceph/blob/master/README.md#run-a-ceph-command

```
kubectl rook-ceph ceph status
kubectl rook-ceph operator restart
kubectl rook-ceph rook version
kubectl rook-ceph ceph versions
```


# Interactive Toolbox

The rook toolbox can run as a deployment in a Kubernetes cluster where you can connect and run arbitrary Ceph commands.
Ref: https://rook.io/docs/rook/latest/Troubleshooting/ceph-toolbox/

```
kubectl create -f deploy/examples/toolbox.yaml
kubectl -n rook-ceph rollout status deploy/rook-ceph-tools
kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- bash
```

# Import External ceph cluster or rook cluster 

Ref: https://rook.io/docs/rook/latest/CRDs/Cluster/external-cluster/external-cluster/

An external cluster is a Ceph configuration that is managed outside of the local K8s cluster. The external cluster could be managed by cephadm, or it could be another Rook cluster that is configured to allow the access (usually configured with host networking).

# Tracing volume created by pod  on  rook ceph cluster.
 
# Check PVC claim used in pod

POD="<POD-NAME>"

```
PVC=`kubectl get pod $POD  -o jsonpath='{.spec.volumes[*].persistentVolumeClaim.claimName} {"\n"}'`

```

# Get pvc details

PV=`kubectl get pvc $PVC -o jsonpath='{.spec.volumeName} {"\n"}'`


# Get pv details

IMAGE=`kubectl get pv $VM -o jsonpath='{.items[*].spec.csi.volumeAttributes.imageName}{"\n"}'`
echo $IMAGE

# check volume in ceph cluster using ceph plugin with rbd
```
root@mon1:~# kubectl rook-ceph rbd ls
poolName        imageName                                     namespace
--------        ---------                                     ---------
ceph-blockpool  csi-vol-aca2ee88-e063-49ab-93be-81f9f1f1b90a  ---
root@mon1:~#
```

# Using multus as network provider for ceph cluster:

rook-ceph cluster provids option to use multus network provider, which allow kubernetes  CNI can be used for public and cluster network , this allow in maintaining strong isolation and impropved performance.

to make use of multus we need to configure it on kubernetes cluster. In case of ceph cluster , 2 NIC will be configured to as resource network-attachment-defination using whereabouts as IPAM for address assignement accross cluster (host-local will not be used here, as it provide IP locally in host only and  create ip collision).

## MULTUS Installation:

Ref: https://github.com/k8snetworkplumbingwg/multus-cni/blob/master/docs/quickstart.md

```
kubectl apply -f https://raw.githubusercontent.com/k8snetworkplumbingwg/multus-cni/master/deployments/multus-daemonset-thick.yml
```

validate installation:

```
kubectl get pods --all-namespaces | grep -i multus
```

## whereabouts installation

An IP Address Management (IPAM) CNI plugin that assigns IP addresses cluster-wide.

If you need a way to assign IP addresses dynamically across your cluster -- Whereabouts is the tool for you. If you've found that you like how the host-local CNI plugin works, but, you need something that works across all the nodes in your cluster (host-local only knows how to assign IPs to pods on the same node) -- Whereabouts is just what you're looking for.

Ref: https://github.com/k8snetworkplumbingwg/whereabouts

```
git clone https://github.com/k8snetworkplumbingwg/whereabouts && cd whereabouts
kubectl apply \
    -f doc/crds/daemonset-install.yaml \
    -f doc/crds/whereabouts.cni.cncf.io_ippools.yaml \
    -f doc/crds/whereabouts.cni.cncf.io_overlappingrangeipreservations.yaml
```
validate whereabouts 

```
kubectl get ds -A | grep -i whereabouts
```
Create network-attach-defination resource using whereabouts IPAM plugin

Whereabouts is particularly useful in scenarios where you're using additional network interfaces for Kubernetes. A NetworkAttachmentDefinition custom resource can be used with a CNI meta plugin such as Multus CNI to attach multiple interfaces to your pods in Kubernetes.

In short, a NetworkAttachmentDefinition contains a CNI configuration packaged into a custom resource. Here's an example of a NetworkAttachmentDefinition containing a CNI configuration which uses Whereabouts for IPAM:

```
apiVersion: "k8s.cni.cncf.io/v1"
kind: NetworkAttachmentDefinition
metadata:
  name: public-conf
spec:
  config: '{
      "cniVersion": "0.3.0",
      "name": "whereaboutsexample",
      "type": "macvlan",
      "master": "eth1",    ### Interface to be used for public network
      "mode": "bridge",
      "ipam": {
        "type": "whereabouts",
        "range": "192.168.112.225/28"
      }
    }'
```


```
apiVersion: "k8s.cni.cncf.io/v1"
kind: NetworkAttachmentDefinition
metadata:
  name: cluster-conf
spec:
  config: '{
      "cniVersion": "0.3.0",
      "name": "whereaboutsexample",
      "type": "macvlan",
      "master": "eth2",    ### Interface to be used for cluster network
      "mode": "bridge",
      "ipam": {
        "type": "whereabouts",
        "range": "192.168.113.225/28"
      }
    }'
```


