Installation

Ref: https://rook.io/docs/rook/latest/Getting-Started/quickstart/



Requirement: 

1. kubernets cluster with at least 3 nodes (kubernetes masters can be used as workers after removing taints)
2. A disk of attached to each nodes to be used as OSD.  Here raw disk is used.
3. each node should have a PUBLIC and Cluster network with IP assigned shouble be able to communicate with each other.
4. Public network will be used to access ceph cluster services.
5. Cluster network will be used by ceph cluster for internal operations like, recovery, balancing of data etc.
6. If re-installing ceph , please make sure to clear directory /var/lib/rook . otherwise it will not allow mon to joing cluster.


Clone rook-ceph git repository

```
git clone --single-branch --branch master https://github.com/rook/rook.git

cd rook/deploy/example
```

Install rook-ceph operators

```
kubectl create -f crds.yaml -f common.yaml -f operator.yaml

# verify the rook-ceph-operator is in the `Running` state before proceeding

kubectl -n rook-ceph get pod
```

There are few setting that need to be verified before we install operator 

ROOK_LOG_LEVEL: "INFO"
ROOK_ENABLE_DISCOVERY_DAEMON: "false"
CSI_CLUSTER_NAME: "my-prod-cluster"



Installing Ceph cluster

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

Get Cluster status:
```
kubectl -n rook-ceph get pod
kubectl get cephclusters.ceph.rook.io -n rook-ceph
```

Install kubectl Plugin

Download and install krew on Linux


Ref: https://krew.sigs.k8s.io/docs/user-guide/setup/install/  OR Install using kubespary whie created cluster or re-run cluster.yaml with tag krew

Install Rook plugin

```

kubectl krew install rook-ceph

```

Test krew plugin 

Ref: https://github.com/rook/kubectl-rook-ceph/blob/master/README.md#run-a-ceph-command

```
kubectl rook-ceph ceph status
kubectl rook-ceph operator restart
kubectl rook-ceph rook version
kubectl rook-ceph ceph versions
```


Interactive Toolbox

The rook toolbox can run as a deployment in a Kubernetes cluster where you can connect and run arbitrary Ceph commands.
Ref: https://rook.io/docs/rook/latest/Troubleshooting/ceph-toolbox/

```
kubectl create -f deploy/examples/toolbox.yaml
kubectl -n rook-ceph rollout status deploy/rook-ceph-tools
kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- bash
```


Import External ceph cluster or rook cluster 

Ref: https://rook.io/docs/rook/latest/CRDs/Cluster/external-cluster/external-cluster/







