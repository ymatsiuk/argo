# Run

```
kind create cluster
helm install argocd argo/argo-cd -f values.yaml
kubectl apply -f root.yaml
```
