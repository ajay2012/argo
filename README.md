# argo

Deployment of argo CD


Ref: https://argo-cd.readthedocs.io/en/stable/getting_started/


Install Argo CD¶

```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Validate all pods are running

```
kubectl get pods -n argocd

```

Get argocd admin password

```
kubectl get secrets -n argocd argocd-initial-admin-secret -o jsonpath='{.data.password}'
```

Create nginx ingress to access the argocd server

```
cat <<EOF > argocd-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: argocd-server-ingress
  namespace: argocd
  annotations:
    cert-manager.io/cluster-issuer: selfsigned-cluster-issuer   #Clusterissuer with selfsigned 
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
    # If you encounter a redirect loop or are getting a 307 response code
    # then you need to force the nginx ingress to connect to the backend using HTTPS.
    #
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  ingressClassName: nginx
  rules:
  - host: argocd.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: argocd-server
            port:
              name: https
  tls:
  - hosts:
    - argocd.example.com
    secretName: argocd-server-tls #Will be created  as expected by argocd-
  EOF
```

```
kubectl apply -f argocd-ingress.yaml
kubectl describe ingress -n argocd argocd-server-ingress
```

To access argocd using CLI install cli tool

Ref: https://argo-cd.readthedocs.io/en/stable/cli_installation/

Install latest stable version

```
VERSION=$(curl -L -s https://raw.githubusercontent.com/argoproj/argo-cd/stable/VERSION)
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/download/v$VERSION/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64
```

connect to argocd server using CLI

```
SVCIP=`kubectl get svc -n argocd argocd-server -o jsonpath='{.spec.clusterIP}'`
argocd login $SVCIP

```




Update argocd deployment to run in insecure mode so that can be access using port 80 via ingress controller

or you can make ingress server service as loadbalancer type and access it.

Deployment access argocd server using inress:
https://argo-cd.readthedocs.io/en/stable/operator-manual/ingress/

delete a stuck namespace in terminating state

https://www.ibm.com/docs/en/cloud-private/3.2.0?topic=console-namespace-is-stuck-in-terminating-state





roo-ceph

# -- Whether to create all Rook pods to run on the host network, for example in environments where a CNI is not enabled
enforceHostNetwork: false

# -- Enable host networking for CSI CephFS and RBD nodeplugins. This may be necessary
  # in some network configurations where the SDN does not provide access to an external cluster or
  # there is significant drop in read/write performance
  enableCSIHostNetwork: true

  # -- If true, run rook operator on the host network
useOperatorHostNetwork:



 # Whether to create all Rook pods to run on the host network, for example in environments where a CNI is not enabled
  ROOK_ENFORCE_HOST_NETWORK: "true"


 # Uncomment it to run rook operator on the host network
      #hostNetwork: true


      # Set to true to enable host networking for CSI CephFS and RBD nodeplugins. This may be necessary
 # in some network configurations where the SDN does not provide access to an external cluster or
 # there is significant drop in read/write performance.
 # CSI_ENABLE_HOST_NETWORK: "true"



 root@ceph-mon01:~/rook/deploy/examples# #python3 create-external-cluster-resources.py --rbd-data-pool-name replicapool --cephfs-filesystem-name myfs --namespace rook-ceph --format bash
root@ceph-mon01:~/rook/deploy/examples# python3 create-external-cluster-resources.py --rbd-data-pool-name replicapool --cephfs-filesystem-name myfs --namespace rook-ceph --format bash
export ARGS="[Configurations]
namespace = rook-ceph
rgw-pool-prefix = default
format = bash
cephfs-filesystem-name = myfs
cephfs-metadata-pool-name = myfs-metadata
cephfs-data-pool-name = myfs-replicated
rbd-data-pool-name = replicapool
"
export NAMESPACE=rook-ceph-external
export ROOK_EXTERNAL_FSID=e94377bc-0ce6-47be-95db-4a3c5a48d39c
export ROOK_EXTERNAL_USERNAME=client.healthchecker
export ROOK_EXTERNAL_CEPH_MON_DATA=d=192.168.122.147:6789
export ROOK_EXTERNAL_USER_SECRET=AQBXR2xnOLVrJBAAyio1EL4gmklEzSQnHas22Q==
export ROOK_EXTERNAL_DASHBOARD_LINK=https://192.168.122.147:8443/
export CSI_RBD_NODE_SECRET=AQBXrmpnxK21DxAAm7nIPRUttyYRv2WGVABHjg==
export CSI_RBD_NODE_SECRET_NAME=csi-rbd-node
export CSI_RBD_PROVISIONER_SECRET=AQBWrmpnCpUpNBAA04wFwtNpHGVbBNuwR9DMBg==
export CSI_RBD_PROVISIONER_SECRET_NAME=csi-rbd-provisioner
export CEPHFS_POOL_NAME=myfs-replicated
export CEPHFS_METADATA_POOL_NAME=myfs-metadata
export CEPHFS_FS_NAME=myfs
export CSI_CEPHFS_NODE_SECRET=AQBYrmpnKTGmAhAAfpMEH8X1zmVL2LAysPfz1w==
export CSI_CEPHFS_PROVISIONER_SECRET=AQBXrmpnO+21JRAAumOlSPbLdkSahJaiR9oEDg==
export CSI_CEPHFS_NODE_SECRET_NAME=csi-cephfs-node
export CSI_CEPHFS_PROVISIONER_SECRET_NAME=csi-cephfs-provisioner
export MONITORING_ENDPOINT=192.168.122.147
export MONITORING_ENDPOINT_PORT=9283
export RBD_POOL_NAME=replicapool
export RGW_POOL_PREFIX=default

root@ceph-mon01:~/rook/deploy/examples#