# priobike-k8s-deployment

## Getting started

Resolve dependencies:
```bash
helm dependency update
```

Install/upgrade: 
```bash
helm upgrade --install priobike-k8s-deployment .
```

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
  - containerPort: 1883 # mqtt/tcp 
    hostPort: 1883
    protocol: TCP
EOF
```

Add ingress NGINX:
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

Wait until the ingress is set up:
```bash
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```
