# Setup and configure resolvconf 

## Install
```bash
sudo apt-update
sudo apt install resolvconf
```

## Set resolveconf config file name servers

```bash
sudo nano /etc/resolvconf/resolv.conf.d/head ## and insert the following

nameserver {{ipaddress}}  ## replace {{ipaddress}} with your dns name server. Add an identical line for each name server. Recommended two.
```

## Enable and start resolvconf
```bash
sudo systemctl start resolvconf.service
sudo systemctl enable resolvconf.service
sudo resolvconf -u
```
