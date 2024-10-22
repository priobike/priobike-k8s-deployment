# priobike-k8s-deployment

## Usage

Resolve dependencies:
```bash
helm dependency update
```

Install/upgrade: 
```bash
helm upgrade --install priobike-k8s-deployment . --namespace priobike-k8s-deployment 
```

> [!IMPORTANT]  
> This backend setup uses another service ([priobike-predictor](https://github.com/priobike/priobike-predictor)) for predictions than our latest app version.
> Therefore, this backend only works with the [priobike-flutter-app](https://github.com/priobike/priobike-flutter-app/) until commit [318eadaa4662404919f3d3c122a24dae230fdbb7](https://github.com/priobike/priobike-flutter-app/tree/318eadaa4662404919f3d3c122a24dae230fdbb7).

## Local development

We use *kind* for our local development cluster. To make Kubernetes Ingress and NodePort working, we need special configuration.
We need to add port mappings for every service/ingress that we want to access from the outside (eg 80,443,1883).

Install kind:
```bash
brew install kind
```

Create cluster: 
```bash
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80 # http
    hostPort: 80
    protocol: TCP
  - containerPort: 443 # https
    hostPort: 443
    protocol: TCP
  - containerPort: 30000 # mqtt/tcp 
    hostPort: 30000 
    protocol: TCP
EOF
```

Add ingress NGINX:
```bash
kubectl apply -f nginx-ingress-deploy.yaml
```

Wait until the ingress is set up:
```bash
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```

Create the namespace if not present yet:
```bash
kubectl create namespace priobike-k8s-deployment
```

Add your registry credentials as secret:
```bash
kubectl create secret docker-registry regcred \
--docker-server=REGISTRY_URL \
--docker-username=USERNAME \
--docker-password=PASSWORD \
--namespace priobike-k8s-deployment
```
