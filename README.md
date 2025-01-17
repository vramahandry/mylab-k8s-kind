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

## Exposing your service through a loadbalancer
You can check [cloud-provider-kind](https://github.com/kubernetes-sigs/cloud-provider-kind?tab=readme-ov-file#install) and his usage on kind [docs](https://kind.sigs.k8s.io/docs/user/loadbalancer/)

```bash
# get the loadalancer IP

LB_IP=$(kubectl get services \
   --namespace ingress-controller \
   ingress-controller-nginx-controller \
   --output jsonpath='{.status.loadBalancer.ingress[0].ip}')
```
### Change dns resolution to use cloud-provider-kind with dnsmasq
#### With brew
```bash
brew install dnsmasq
```
Change the dnsmasq config in `$(brew -prefix)/etc/dnsmasq.conf`<br>
Uncomment `conf-dir=/opt/homebrew/etc/dnsmasq.d/,*.conf`in the end of file

Add your domain name like this
```bash
echo "address=/kind.cluster/<IP_CLOUD_PROVIDER_KIND>" | sudo tee $(brew --prefix)/etc/dnsmasq.d/kind.k8s.conf
```
Start/Restart the service dnsmasq
```bash
sudo brew services restart dnsmasq
```
Check the status
```bash
sudo brew services list | grep dnsmasq
```
Add dns resolver for your domain
```bash
sudo mkdir -v /etc/resolver

sudo bash -c 'echo "nameserver 127.0.0.1" > /etc/resolver/kind.cluster'
```

Now you can access to `*.kind.cluster` domains
