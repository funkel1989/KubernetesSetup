# Kubernetes Setup and Installation

## Server Prep

- Plan out IP ranges required for your network. All nodes should require static IP's set
- Install Ubuntu 20.04. Include open SSH and etcd in install
  - Drive should not be setup as lvm
- Update and upgrade your installation of Ubuntu
- Install the following dependencies
  - nfs-common
  - open-iscsi
  - [resolve.conf]()

## Dependencies
- [Setup Mysql Database](./install-and-setup-mysqldb.md)
- [Setup k3s load balancer using nginx](./configure-nginx-k3s-lb.md)