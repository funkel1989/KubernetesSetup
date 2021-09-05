# Install and configure MetalLB for Kubernetes k3s

```bash
## add and update the metalLB Repo
helm repo add metallb https://metallb.github.io/metallb
helm repo update

## Create a kubernetes namespace to install metalLB on
kubectl create namespace metallb-system

## Install Metallb in that namespace specifying the config in the included yaml file below
helm install metallb metallb/metallb -n metallb-system

## Apply a metallb config map using kubectl. config map found in this repo at below path
kubectl apply -f helmCharts/metallb/metallb-config.yaml
```
