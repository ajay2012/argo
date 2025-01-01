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


Kubernete's kubete bootstrapping process
https://kubernetes.io/docs/reference/access-authn-authz/kubelet-tls-bootstrapping/



https://kubernetes.io/docs/tasks/tls/