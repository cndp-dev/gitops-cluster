# gitops-cluster

## Bootstrap
### Cluster creation

```
gcloud container clusters create gitops --num-nodes 1 --machine-type e2-medium --disk-size 30 --enable-autoscaling --min-nodes 1 --max-nodes 3 --preemptible --enable-dataplane-v2
gcloud container clusters get-credentials gitops
```

### Initial setup of ArgoCD & Crossplane

```
kubectl apply -k bootstrap/argocd
kubectl wait --for=condition=ready pod -l app.kubernetes.io/name=argocd-server --namespace argocd --timeout=300s

argocd admin initial-password -n argocd
kubectl port-forward -n argocd --address='0.0.0.0' service/argocd-server 8080:80
argocd login --insecure --username admin --password <PASSWORD> 0.0.0.0:8080
argocd account update-password --current-password <PASSWORD>
kubectl delete -n argocd secrets argocd-initial-admin-secret

kubectl apply -f bootstrap/crossplane.yaml
```

### Configure ArgoCD Provider

```
kubectl port-forward -n argocd --address='0.0.0.0' service/argocd-server 8080:80
ARGOCD_ADMIN_SECRET=<PASSWORD>
ARGOCD_ADMIN_TOKEN=$(curl -s -X POST -k -H "Content-Type: application/json" --data '{"username":"admin","password":"'$ARGOCD_ADMIN_SECRET'"}' https://localhost:8080/api/v1/session | jq -r .token)
ARGOCD_API_TOKEN=$(curl -s -X POST -k -H "Authorization: Bearer $ARGOCD_ADMIN_TOKEN" -H "Content-Type: application/json" https://localhost:8080/api/v1/account/provider-argocd/token | jq -r .token)
kubectl create secret generic argocd-credentials -n crossplane-system --from-literal=authToken="$ARGOCD_API_TOKEN"
```
