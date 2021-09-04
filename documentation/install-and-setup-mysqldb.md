# Install and setup Mysql DB (single instance)

## Dependencies
- Install Ubuntu server 20.04


## MySql Install and Setup
```bash
sudo apt update
sudo apt install mysql-server
sudo mysql_secure_installation
```
- Answer the following setup questions as you see fit. Consider defaults for highest potential security
- Set password for root user when asked
- Set mysql config file to allow remove access to the db
```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```
- Comment out the line that says bind-address then close and save the file
```bash
sudo systemctl restart mysql
sudo ufw allow 3306
```

## Setup User accounts and create your needed databases
- Create a admin user account for you with the following commands
    - NOTE: the following commands assume your DB will be accessable outside of just localhost
```sql
CREATE USER 'username'@'%' IDENTIFIED BY 'password';
GRANT PRIVILEGES ON *.* TO 'username'@'%';
FLUSH PRIVILEGES;
```
- Create database required from your k3s install
```sql
CREATE DATABASE kubernetes;
```
- Create a Kubernetes user for your install of k3s in HA
```sql
CREATE USER 'kubernetes'@'%' IDENTIFIED BY 'password';
GRANT PRIVILEGES ON kubernetes.* TO 'kubernetes'@'%';
FLUSH PRIVILEGES;
exit
```

- Test that you can access your database using mysql workbench or other preferred software on both user accounts that you setup from another machine on your network.