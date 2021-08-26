# Preface

This guide has been made from information condensed down from the following documentation:


https://devdocs.magento.com/guides/v2.4/install-gde/prereq/nginx.html#install-nginx

https://devdocs.magento.com/guides/v2.4/install-gde/prereq/mysql.html

https://devdocs.magento.com/guides/v2.4/install-gde/prereq/connect-auth.html

https://www.rosehosting.com/blog/how-to-install-magento-2-4-with-lemp-stack-on-ubuntu-20-04/

# Introduction

This document covers Magento and how I'm getting it installed on an Amazon EC2 instance
## Prerequisites

Before beginning the installation process, you must create an account at https://magento.com/

After creating an account, acquire your public and private keys, needed for a later step, by visiting [this](https://marketplace.magento.com/) site. Once on there, click on your name in the top right corner and then click on "My Profile". Once there, click on "Access Keys", located under Marketplace > My Products.  

Assuming you don't already have an access key created, simply create one and name it something appropriate.  Once created, make note that in a later step, your public key will act as your username and your private key will act as your password.

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
	4. Uncomment `;extension=curl`
3. Save the changes you made and restart PHP with `systemctl restart php7.4-fpm`

### Install MariaDB
1. `sudo apt install mariadb-server`
2. `sudo mysql_secure_installation`
This will launch the configurator for mariaDB
```
Enter current password for root (enter for none): Press [Enter] since no password is set by default
Set root password? [Y/n]: Y (Keep this password handy for a later step)
Remove anonymous users? [Y/n]: Y
Disallow root login remotely? [Y/n]: Y
Remove test database and access to it? [Y/n]: Y
Reload privilege tables now? [Y/n]: Y
```
*Note* - The credentials created in this process will need to be put into AWS Secrets Manager.

Once done:
3. `systemctl status mariadb.service`
4. `systemctl enable mariadb.service`

### Configure DB
1. `sudo mysql -u root -p`
2. Enter the `magento` user password when asked
3. Enter the following commands in the order shown to create a database instance named magento with username magento:
```
create database magento;
create user 'magento'@'localhost' IDENTIFIED BY 'Password';
GRANT ALL ON magento.* TO 'magento'@'localhost';
flush privileges;
```
4. `mysql -u magento -p`
5. Use the root password we set up in the second SQL statement referenced as `Password`
6. `use magento;` and ensure the output is `Database changed`

### Install Elastic Search
1. `wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.6.1-amd64.deb`
2. `sudo dpkg -i elasticsearch-7.6.1-amd64.deb`
3. `sudo systemctl start elasticsearch`
4. `sudo systemctl status elasticsearch`
5. `curl -XGET 'http://localhost:9200'`

### Configure Elastic Search
1. `sudo vim /etc/elasticsearch/elasticsearch.yml`
- `cluster.name: Magento Cluster`
- `node.name: Magento Node`
- `network.host: localhost`
2. `systemctl restart elasticsearch.service`
3. Re-run `curl -XGET 'http://localhost:9200'` to confirm changes are reflected

### Permissions
1. `sudo usermod -g www-data magento`
2. `sudo chown -R magento:www-data /var/www/html/`

### Installing PHP Extensions
At this point, I found that before I could run the `composer` command in the next section, I had several php extensions I had to install. Those were:
1. `sudo apt install php-bcmath`
2. `sudo apt install php7.4-curl`
3. `sudo apt install php-xml`
4. `sudo apt install php-7.4-mbstring`
5. `sudo apt install php7.4-intl`
6. `sudo apt install php7.4-gd`
7. `sudo apt install php7.4-mysql`
8. `sudo apt install php7.4-soap`


### Installing Magento
1. `sudo apt install zip unzip php-zip`
2. `cd /var/www/html` and ensure the directory is empty of any contents. Generally, Nginx have a test file in here that you will need to delete.
3. Install composer globally: `curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/bin --filename=composer`
4. `composer create-project --repository=<https://repo.magento.com/ magento/project-community-edition .`
5. When prompted, provide public key as username and private key as password
   - https://marketplace.magento.com/customer/accessKeys/

Once finished, you may be presented with some messages highlighted in yellow. These are not necessarily anything to worry about unless they're explicitly warnings or if the highlighting is red instead of yellow.

We now run the composer install command to actually Magento, itself
```
composer install
bin/magento setup:install \
--base-url=<http://localhost/magento2ee> \
--db-host=localhost \
--db-name=magento \
--db-user=magento \
--db-password=db_password \
--backend-frontname=admin \
--admin-firstname=Tanner \
--admin-lastname=Cude \
--admin-email=email \
--admin-user=magentoadmin \
--admin-password=alphanumberic_password \
--language=en_US \
--currency=USD \
--timezone=America/Chicago \
--use-rewrites=1
```
Note that this step may take several minutes to complete but afterward, Magento will have been installed and you will then need to configure Nginx
