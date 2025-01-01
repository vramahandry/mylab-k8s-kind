# mylab-k8s-kind

## Installation of the root cluster

```bash
kind create cluster --config kind/kind-mylab-config.yaml
```

## Installation of helm charts
In this order:

- cilium
```bash
helm upgrade --install cilium ./helm/cilium -n kube-system
```
- argocd
```bash
 helm upgrade --install argocd ./helm/argocd/ -n argocd --create-namespace
```
- root-app

It will deploy ArgoCD apps:
- argocd himself
- argocd apps in external repos

```bash
helm upgrade --install root-app-argocd ./helm/root-app -n argocd
```

## ArgoCD access

Get access to the ArgoCD UI

- Port forward service to localhost:8080
```bash
kubectl -n argocd port-forward svc/argocd-server 8080:80
```
- Get the password of ArgoCD UI

```bash
kubectl get -n argocd secret argocd-initial-admin-secret --template={{.data.password}} | base64 -d
```