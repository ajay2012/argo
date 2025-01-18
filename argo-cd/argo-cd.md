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

Create nginx ingress to access the argocd server using selfsigned certificate using cert-manager

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

argocd server can be access without TLS

update deployment args with --insecure like below

```
kubectl edit deployment -n argocd argocd-server

--insecure

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

Add kubernetes cluster to argocd

By default local cluster is registered with argocd and can be check with below command. Other clusters can be added to argo cd so that applications can be installed accross other clusters from one UI/CLI of argocd.

Check current registered k8s cluster

```
argocd cluster list 
```


check other availble clusters, there is magic, you need to check conttext in kubeconfig file add use those context name to add them to argocd server

```
kubectl config get-context
```

add additional clusters

```
argocd cluster add context-name
```

User rm to remove cluster from argocd.

Use of custome values.yaml file

I have add a helm app as submoudle under git repository. And i created its custome values.yaml file under seprate directory so that it is intact. to add this values.yaml file when creating argocd app using UI I didn't find any way to use this values.yaml , how ever with argocd cli I have tested it. see below. After this when ever I make change to "helm-overrides/values.yam"  file , app setting is updated.

```
root@master01:~# argocd app get argocd/testapp
Name:               argocd/testapp
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          default
URL:                https://10.233.58.43/applications/testapp
Source:
- Repo:             https://github.com/ajay2012/lab_setup.git
  Target:           HEAD
  Path:             submodules/argocd-example-apps/helm-guestbook
  Helm Values:      values.yaml     ### Inital values.yaml that is inside that chart under"submodules/argocd-example-apps/helm-guestbook" ###
SyncWindow:         Sync Allowed
Sync Policy:        Manual
Sync Status:        Synced to HEAD (09fa3a9)
Health Status:      Healthy

GROUP  KIND        NAMESPACE  NAME                    STATUS  HEALTH   HOOK  MESSAGE
       Service     default    testapp-helm-guestbook  Synced  Healthy        service/testapp-helm-guestbook created
apps   Deployment  default    testapp-helm-guestbook  Synced  Healthy        deployment.apps/testapp-helm-guestbook created
root@master01:~# argocd app  set argocd/testapp --values ../../helm-overrides/values.yaml
FATA[0001] rpc error: code = InvalidArgument desc = application spec for testapp is invalid: InvalidSpecError: Unable to generate manifests in submodules/argocd-example-apps/helm-guestbook: rpc error: code = Unknown desc = `helm template . --name-template testapp --namespace default --kube-version 1.31 --values <path to cached source>/submodules/helm-overrides/values.yaml <api versions removed> --include-crds` failed exit status 1: Error: open <path to cached source>/submodules/helm-overrides/values.yaml: no such file or directory
root@master01:~# argocd app  set argocd/testapp --values ../../../helm-overrides/values.yaml    #Custome file exist in repository at root "helm-overrides/values.yaml" 
root@master01:~# argocd app get argocd/testapp
Name:               argocd/testapp
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          default
URL:                https://10.233.58.43/applications/testapp
Source:
- Repo:             https://github.com/ajay2012/lab_setup.git
  Target:           HEAD
  Path:             submodules/argocd-example-apps/helm-guestbook
  Helm Values:      ../../../helm-overrides/values.yaml
SyncWindow:         Sync Allowed
Sync Policy:        Manual
Sync Status:        OutOfSync from HEAD (09fa3a9)
Health Status:      Healthy

GROUP  KIND        NAMESPACE  NAME                    STATUS     HEALTH   HOOK  MESSAGE
       Service     default    testapp-helm-guestbook  Synced     Healthy        service/testapp-helm-guestbook created
apps   Deployment  default    testapp-helm-guestbook  OutOfSync  Healthy        deployment.apps/testapp-helm-guestbook created
root@master01:~#
```

Kubernete's kubete bootstrapping process
https://kubernetes.io/docs/reference/access-authn-authz/kubelet-tls-bootstrapping/



https://kubernetes.io/docs/tasks/tls/
