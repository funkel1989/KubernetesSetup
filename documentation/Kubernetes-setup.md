# Kubernetes Setup and Installation

## Server Prep

- Plan out IP ranges required for your network. All nodes should require static IP's set
- Install Ubuntu 20.04. Include open SSH and etcd in install
  - Drive should not be setup as lvm
- Update and upgrade your installation of Ubuntu
- Install the following dependencies
  - nfs-common
  - open-iscsi
  - [resolve.conf](./configure-resolvconf.md)
  - Set esxi configuration needed on kubernetes servers so kubernetes can see virtual drives
    - disk.EnableUUID = TRUE
      - When the virtual machine is powered off
      - Edit settings
        - VM Options
          - Advanced
            - edit configuration
              - Add Parameters
                - Key: disk.EnableUUID
                - Value: TRUE

## Dependencies

- [Setup Mysql Database](./install-and-setup-mysqldb.md)
- [Setup k3s load balancer using nginx](./configure-nginx-k3s-lb.md)

## Installing K3s

### Set environment variables and install master kubernetes nodes

```bash
export K3S_DATASTORE_ENDPOINT='mysql://kubernetes:passwordhere@tcp(192.168.1.30:3306)/kubernetes'

export INSTALL_K3S_VERSION=v1.21.4+k3s1

## install kubernetes with the following command. Replace tls-san ip with the Ip address of previously setup nginx load balancer IP address and token with a random token string you will need to remember for all other master node installs
sudo curl -sfL https://get.k3s.io | sh -s - server --node-taint CriticalAddonsOnly=true:NoExecute --tls-san 192.168.1.40 --write-kubeconfig-mode 644 --disable traefik --disable servicelb --token="tokenHere"

kubectl get nodes  ## use this to test. you should see your node printed out as shown below

NAME      STATUS   ROLES                  AGE   VERSION
{{nodeHostName}}   Ready    control-plane,master   75s   v1.21.4+k3s1
```

## Install Agent Node

- Get token from master node by running the following command on your kubernetes master node

```bash
sudo cat /var/lib/rancher/k3s/server/node-token
```

- install kubernetes agent server

```bash
sudo curl -sfL https://get.k3s.io | K3S_URL=https://192.168.1.40:6443 K3S_TOKEN=somerandomtokenhere sh -

## Test by running kubectl get nodes from your master server

sudo kubectl get nodes

## you should see the below if it works

NAME      STATUS   ROLES                  AGE   VERSION
k3s-m-1   Ready    control-plane,master   16m   v1.21.4+k3s1
k3s-a-1   Ready    <none>                 21s   v1.21.4+k3s1

```

## Setup Kubectl on development Machine

- cat out your k3s.yaml file from a control plane node

```bash
sudo cat /etc/rancher/k3s/k3s.yaml
```

- Copy the file and paste into your .kube/config file on your developer machine
- Replace the Ip address set to 127.0.0.1:6443 with your nginx load balancer ip address

- test a kubectl get nodes from your development machine

## Installing MetalLB

- Follow these instructions to [Install Metallb](./install-configure-metallb.md)

## Install Rancher

- Follow these instructions to [Install Rancher](./install-configure-rancher.md)

## Install Longhorn

- NOTE: It is best to create another k3s agent server that will act as the dedicated storage server. Set this up now. It can be configured exactly like the pervious agent server.
- NOTE: The longhorn UI seems to be really slow

- While setting up longhorn I went the route of using the rancher app catalog. This repo includes a values file required (at least during my last install it was required) if you want to setup taints but generally after installation I use the UI to disable nodes not used for storage so long horn only uses those roles.

- UI allows the setting up a taint and I utilize the below settings
  - Longhorn UI: StorageOnly=true:NoExecute;CriticalAddonsOnly=true:NoExecute
  - Commands to execute against my nodes
    ```bash
    kubectl taint nodes k3s-storage-1 CriticalAddonsOnly=true:NoExecute
    kubectl taint nodes k3s-storage-1 StorageOnly=true:NoExecute
    ```

## Install Traefik 2

- NOTE: Traefik is being setup to receive lets encrypt certs with a dns challenge set to route 53. The traefik config includes the secret needed for this challenge along with a config map and persistent claim. Set this up first before you do anything else.
- NOTE: When applying the persistent storage claim longhorn has to be set as default for it to be used and it must be the only default. The current install of Rancher
  creates a local-path and has a pod that consistently will re-create this and set it as default. I normally get my command ready to apply for my helm chart, delete the local-path hit enter and more times then not I will be fast enough to get my persistent claim added before local-path comes back. Still looking for a better resolution to this problem as longhorn should always be the default.

- Setup helm to install traefik

```bash
helm repo add traefik https://helm.traefik.io/traefik
helm repo update
```

- Apply traefik config yaml

```bash
kubectl apply -f helmCharts/traefik/traefik-config.yaml
```

- Install traefik using help and apply the traefik-values.yaml to the helm install command

```bash
helm install traefik traefik/traefik --namespace=traefik-system --values=helmCharts/traefik/traefik-values.yaml
```

- Install traefik ingress dashboard

```bash
kubectl apply -f helmCharts/traefik/traefik-dashboard-ingressroute.yaml
```