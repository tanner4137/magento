This guide has been made from information condensed down from the following documentation:

https://devdocs.magento.com/guides/v2.4/install-gde/prereq/nginx.html#install-nginx

https://devdocs.magento.com/guides/v2.4/install-gde/prereq/mysql.html

https://devdocs.magento.com/guides/v2.4/install-gde/prereq/connect-auth.html

# Introduction

This document covers Magento and how I'm getting it installed on an Amazon EC2 instance

## Installation steps

### Create `magento` user
1. `sudo adduser magento`
2. `sudo usermod -aG sudo magento`
	1. Adds user to the `sudo` group
3. `sudo -iu magento`
	1. Logs in as `magento`

### Install nginx
1. `sudo apt -y install nginx`

### Install PHP
1. `apt-get -y install php7.4-fpm`
2. Might possibly need to run `apt-get -y install php7.4-cli`

### Configure PHP
1. Open the `php.ini` files in an editor:
	1. `vim /etc/php/7.4/fpm/php.ini`
	2. `vim /etc/php/7.4/cli/php.ini`
2. Edit both to include the following values:
	1. `memory_limit = 2G`
	2.	`max_execution_time = 1800`
	3. `zlib.output_compression = On`
3. Save the changes you made and restart PHP with `systemctl restart php7.4-fpm`

### Install MariaDB
1. `sudo apt install mariadb-server`
2. `sudo mysql_secure_installation`
This will launch the configurator for mariaDB
```
Enter current password for root (enter for none): Press [Enter] since no password is set by default
Set root password? [Y/n]: N (You can set a password if you like)
Remove anonymous users? [Y/n]: Y
Disallow root login remotely? [Y/n]: Y
Remove test database and access to it? [Y/n]: Y
Reload privilege tables now? [Y/n]: Y
```
Once done:
3. `systemctl status mariadb.service`
4. `systemctl enable mariadb.service`

