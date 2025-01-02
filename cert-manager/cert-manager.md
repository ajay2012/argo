#cert-manager is a powerful and extensible X.509 certificate controller for Kubernetes and OpenShift workloads. It will obtain certificates from a variety of Issuers, both popular public Issuers as well as private Issuers, and ensure the certificates are valid and up-to-date, and will attempt to renew certificates at a configured time before expiry.


#Installtion:

##Cert-manager:

Ref: https://cert-manager.io/docs/installation/helm/

```
helm repo add jetstack https://charts.jetstack.io --force-update
```

```
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.16.2 \
  --set crds.enabled=true
 ```

More setting

Enable Gateway API support:
```
--set config.enableGatewayAPI=true

```
Disable Promethus and change webhook:
```
  --set prometheus.enabled=false \  # Example: disabling prometheus using a Helm parameter
  --set webhook.timeoutSeconds=4   # Example: changing the webhook timeout using a Helm parameter
```

##cert-manger client utility

Ref: https://cert-manager.io/docs/reference/cmctl/#installation

Installation:

```
OS=$(go env GOOS); ARCH=$(go env GOARCH); curl -fsSL -o cmctl https://github.com/cert-manager/cmctl/releases/latest/download/cmctl_${OS}_${ARCH}
chmod +x cmctl
sudo mv cmctl /usr/local/bin
```

Check cert-manager api status

```
cmctl check api
cmctl check api --wait=2m
```

can be used to approve and deny certs as well, check help for more functions.

```
cmctl approve -n <namespace> <certificate>
cmctl deny -n <namespace> <certificate>
```





