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