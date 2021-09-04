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


- [Install Metallb]()