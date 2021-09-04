# Installing Nginx as a simple load balancer

## Pre-req

- Install ubuntu 20.04
- Update and upgrade your server

## Install and enable nginx base install

```bash
sudo apt-get install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
```

## Configure Nginx for Load Balancing Kubernetes Cluster

```bash
sudo rm -rf /etc/nginx/sites-enabled/default
sudo nano /etc/nginx/nginx.conf
```

Delete everything from the nginx.conf file and paste the data from [this conf file](../configurationFiles/nginx.rancher-k3s-only.conf)

- Restart the nginx service and then test it
```bash
sudo systemctl restart nginx
sudo nginx -t

## The below should be displayed
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
