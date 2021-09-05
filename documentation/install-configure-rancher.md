# Install and configure rancher for k3s

- Add rancher helm repo

```bash
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable

## update repos after adding
helm repo update
```

- Create a rancher namespace. Use the generic name rancher provides of cattle-system for ease

```bash
kubectl create namespace cattle-system
```

- Install cert manager which is required for rancher

```bash
## cert manager dependencies
kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.2.0/cert-manager.crds.yaml

## cert manager namespace
kubectl create namespace cert-manager

## helm repo for cert manager
helm repo add jetstack https://charts.jetstack.io
helm repo update

## Install Cert Manager
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --version v1.5.3

## Check rollout status. Do not continue until all pods are completely up
kubectl -n cattle-system rollout status deploy/rancher
```

- Install rancher on your cluster using helm

```bash
helm install rancher rancher-latest/rancher \
  --namespace cattle-system \
  --set hostname=rancher.mycavanaughnetwork.com \
  --version 2.6.0
```

- Check deployment status with:
```bash
kubectl get pods -n cattle-system
```

- Expose rancher for testing and basic utilization through metallb
```bash
kubectl expose deployment rancher -n cattle-system --type=LoadBalancer --name=rancher-lb --port=443
```
- Either set your hosts file metallb ip address to rancher.mycavanaughnetwork.com or create a dns cname/a record to point to the metallb ip address
    - we will undo this mapping at a later time(maybe?)

- NOTE: At this point rancher told me for first time access i had to get a bootstrap password and I didn't set one during install. It provides you with a command to get the bootstrap password and when I did this it returned nothing. It seems if you do a helm install and a helm uninstall sometimes junk can get left behind and screw this process up. The following commands reset the admin user and provided me with a new password.

```bash
## set kubeconfig file to env var to make these commands a bit shorter
KUBECONFIG=/etc/rancher/k3s/k3s.yaml

## Reset the admin users for rancher
kubectl --kubeconfig $KUBECONFIG -n cattle-system exec $(kubectl --kubeconfig $KUBECONFIG -n cattle-system get pods -l app=rancher | grep '1/1' | head -1 | awk '{ print $1 }') -- ensure-default-admin

## Reset and provide the password of the above default user
kubectl --kubeconfig $KUBECONFIG -n cattle-system exec $(kubectl --kubeconfig $KUBECONFIG -n cattle-system get pods -l app=rancher | grep '1/1' | head -1 | awk '{ print $1 }') -- ensure-default-admin
```

- At this point I was able to access rancher through my exposed URL
