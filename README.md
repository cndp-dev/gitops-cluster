# gitops-cluster

```
kubectl apply -k bootstrap/argocd
kubectl wait --for=condition=ready pod -l app.kubernetes.io/name=argocd-server --namespace argocd --timeout=300s

argocd admin initial-password -n argocd
kubectl port-forward -n argocd --address='0.0.0.0' service/argocd-server 8080:80
argocd login --insecure --username admin --password PASSWORD 0.0.0.0:8080
argocd account update-password --current-password PASSWORD
kubectl delete -n argocd secrets argocd-initial-admin-secret

kubectl apply -f bootstrap/crossplane.yaml
kubectl apply -f ovh-creds.yaml
```

```

```