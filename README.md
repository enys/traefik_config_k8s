# Traefik in k8s
https://doc.traefik.io/traefik/getting-started/install-traefik/
https://traefik.io/blog/install-and-configure-traefik-with-helm/


## Install via Helm
### Add helm char repository
helm repo add traefik https://helm.traefik.io/traefik

### Generate template
```
helm template traefik traefik/traefik --namespace=traefik -f values.yaml >| traefik/template.yaml
```

### Install
```
kubectl create namespace traefik
kubectl apply -f traefik/traefik-config.yaml
helm install traefik traefik/traefik --namespace=traefik --values=traefik/traefik-chart-values.yaml
```

and then when needed to update:
```
helm upgrade traefik traefik/traefik --namespace=traefik --values=traefik/traefik-chart-values.yaml
```

### Traefik dashboard
Creating an ingressroute to access the dashboard:
```
kubectl apply -f traefik/secret-traefik-dashboard.yaml
kubectl apply -f traefik/ingressroute-traefik-dashboard.yaml
```
or via port-forwarding:
```
kubectl port-forward -n traefik $(kubectl get pods -n traefik --selector "app.kubernetes.io/name=traefik" --output=name | head -n 1) 9000:9000
```

# Cert-manager
https://cert-manager.io/docs/installation/kubernetes/

## Install via Helm
### Add helm char repository
```
helm repo add jetstack https://charts.jetstack.io && helm repo update
```

### Install
```
kubectl create namespace cert-manager
helm install --namespace cert-manager cert-manager jetstack/cert-manager --set installCRDs=true
```
Optionaly install:
```
kubectl krew install cert-manager
```

### Check cert-manager

```
kubectl get pods --namespace cert-manager
```

### Install cert-manager-webhook-gandi
clone repository https://github.com/bwolf/cert-manager-webhook-gandi/
or until the prvious one is patched, 
https://github.com/ecolowtech/cert-manager-webhook-gandi/

Install helm chart :
```
helm install  cert-manager-webhook-gandi --namespace cert-manager ./deploy/cert-manager-webhook-gandi --set image.tag=v0.1.1
```

### Gandi secret
Create the secret to keep the Gandi API key in the  namespace, where later on the ClusterIssuer is created:
```
kubectl create -n cert-manager secret generic gandi-credentials \
     --from-literal=api-token='<GANDI-API-KEY>'
```
Grant permission for the service-account to access the secret holding the Gandi API key:
```
kubectl apply -n cert-manager -f rbac.yaml
```
### Check install
```
check :
     kubectl get pods -n cert-manager --watch
     kubectl logs -n cert-manager cert-manager-webhook-gandi-XYZ
```
### Install issuer
```
kubectl apply -f cert-manager/http-le-staging-issuer.yaml
kubectl apply -f cert-manager/dns01-le-staging-issuer.yam
```

### Create example wildcard cert
```
kubectl apply -f cert-manager/default-wildcard-cert.yaml
kubectl cert-manager status certificate wildcard-default-cert
kubectl describe certificate wildcard-default-cert
```
# Oauth

We will use https://github.com/thomseddon/traefik-forward-auth to provide basic oauth login in front of selected services

We have selected to add the container as a sidecar to traefik. See [chart-values](traefik/traefik-chart-values.yaml).

You will need to create the appropriate secrets:
```
kubectl apply -f traefik/traefik-oauth-secret-private.yaml
```

And create the forwardAuth middleware, that will delegate athentication to this new container:
```
kubectl apply -f traefik/traefik-oauth-middleware.yaml
```
