---
title: "KodeKloud's 100 Days of DevOps: Day 11 - 20"
date: 2025-12-31 # YYYY-MM-DD
description: "A walkthrough for days 11 through 20 of KodeKloud's 100 Days of DevOps challenges."
# weight: 1
# aliases: ["/first"]
draft: true
series: ["KodeKloud's 100 Days of DevOps"]
categories: ["DevOps", "KodeKloud"]
tags: ["devops", "linux", "kodekloud"]
showToc: true
TocOpen: false
hidemeta: false
disableHLJS: true # to disable highlightjs
disableShare: false
hideSummary: true
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: false
ShowPostNavLinks: true
ShowWordCount: false
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page
    cover.responsiveImages: true
---

## Day 11: Install and Configure Tomcat Server

## Day 12: Linux Networking Services

## Day 13: IPtables Installation and Configuration

## Day 14: Linux Process Troubleshooting

## Day 15: Setup SSL for Nginx

## Day 16: Install and Configure Nginx as an LBR

## Day 17: Install and Configure PostgreSQL

## Day 18: Configure LAMP Server

Prompt:

> xFusionCorp Industries is planning to host a WordPress website on their infra in Stratos Datacenter. They have already done infrastructure configuration—for example, on the storage server they already have a shared directory /vaw/www/html that is mounted on each app host under /var/www/html directory. Please perform the following steps to accomplish the task:  
> 
> a. Install httpd, php and its dependencies on all app hosts.  
> b. Apache should serve on port 5004 within the apps.  
> c. Install/Configure MariaDB server on DB Server.  
> d. Create a database named kodekloud_db5 and create a database user named kodekloud_cap identified as password B4zNgHA7Ya. Further make sure this newly created user is able to perform all operation on the database you created.  
> e. Finally you should be able to access the website on LBR link, by clicking on the App button on the top bar. You should see a message like App is able to connect to the database using user kodekloud_cap

Okay, this shouldn't be too bad. Let's take stock of what's already on the servers and see what we need to get this running. I'm starting on `stapp01` and will repeat the steps where needed on the other app servers.

First up, what packages are installed? Running:

```bash
dnf list --installed | grep httpd
dnf list --installed | grep php
```
doesn't net us much, just blank lines. We'll need to install both the server and the language. To do that, we run the following:

```bash
$ sudo dnf install httpd php
```

 Next, let's check the app we're deploying:
```bash
[tony@stapp01 ~]$ ls -la /var/www/html
total 12
drwxr-xr-x 2 root root 4096 Jan  1 03:38 .
drwxr-xr-x 3 root root 4096 Jan  1 03:37 ..
-rw-r--r-- 1 root root  266 Jan  1 03:37 index.php
```

Just a single page. Here are the contents of `index.php`:

```php
<?php

$dbname = 'kodekloud_db5';
$dbuser = 'kodekloud_cap';
$dbpass = 'B4zNgHA7Ya';
$dbhost = 'stdb01';

$link = mysqli_connect($dbhost, $dbuser, $dbpass) or die("Unable to Connect to '$dbhost'");
echo "App is able to connect to the database using user $dbuser";
?>
```

Trying to run the file with `sudo php /var/www/html/index.php` produces the error `Fatal error: Call to undefined function mysqli_connect()` - meaning we don't have the MySQL library installed. [After doing some searching](https://stackoverflow.com/questions/25281467/fatal-error-call-to-undefined-function-mysqli-connect), the way to rectify this is to install the `php-mysqli` package (`php-mysqlnd` on Fedora's repositories) using either `yum` or `dnf`.

```bash
$ sudo dnf install php-mysqlnd
```

Next up - we need to set Apache to bind to the correct port so that way the load balancer can reach the server. The default config for `httpd` on Fedora lives at `/etc/httpd/conf/httpd.conf`. You can edit it using Vi, Vim, Nano, or any other text or stream editors you know - I'm using `sed` for repeatability and consistency across the other servers.

```
sudo sed -i 's/Listen 80/Listen 5004/g' /etc/httpd/conf/httpd.conf
sudo systemctl restart httpd
```

After restarting the service, you can verify the changes are in effect by running `systemctl status httpd` or checking the listening ports by using `sudo ss -antp | grep 5004` and making sure everything appears correct. With that, we can move over to the database portion of the setup.

The database first needs installation - confirm that MariaDB isn't already present using `dnf`, and then run the following if it's missing:

```bash
$ sudo dnf install mariadb-server
$ sudo systemctl enable --now mariadb
```

Once it's installed and the service is available, we can start configuring the defaults. Run through the `mysql_secure_installation` script, choosing options as you go along - most of the defaults are fine - and then edit the bind-address in our MariaDB configuration to listen on `0.0.0.0` to accept remote connections.
```
sudo mysql_secure_installation
sudo vi /etc/my.cnf.d/mariadb-server.cnf, uncomment the bind-address line
```

Finally, we need to set up the database user so that our apps can have access to run queries, update tables, etc. The demo site doesn't do any of that, but will just check if the connection is live in the `mysqli_connect` call from the code at the beginning of this section. I chose to limit the IP range to 172.16.X, since that's what the containers were running for both 

```
$ sudo mysql -u root -p
Enter password:
MariaDB [(none)]> CREATE DATABASE kodekloud_db5;
MariaDB [(none)]> CREATE USER 'kodekloud_cap'@'172.16.%' IDENTIFIED BY 'B4zNgHA7Ya';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON kodekloud_db5.* TO 'kodekloud_cap'@'172.16.%';
MariaDB [(none)]> FLUSH PRIVILEGES;
```

Remote connection: https://mariadb.com/docs/server/mariadb-quickstart-guides/mariadb-remote-connection-guide  



## Day 19: Install and Configure Web Application

Prompt:

> xFusionCorp Industries is planning to host two static websites on their infra in Stratos Datacenter. The development of these websites is still in-progress, but we want to get the servers ready. Please perform the following steps to accomplish the task:
>  
> a. Install httpd package and dependencies on app server 3.  
> b. Apache should serve on port 8083.  
> c. There are two website's backups /home/thor/news and /home/thor/cluster on jump_host. Set them up on Apache in a way that news should work on the link http://localhost:8083/news/ and cluster should work on link http://localhost:8083/cluster/ on the mentioned app server.  
> d. Once configured you should be able to access the website using curl command on the respective app server, i.e curl http://localhost:8083/news/ and curl http://localhost:8083/cluster/  

ssh banner@stapp03

sudo sed -i "s/Listen 80/Listen 8083/g" /etc/httpd/conf/httpd.conf
sudo systemctl restart httpd

scp -r news banner@stapp03:/home/banner/news
scp -r cluster banner@stapp03:/home/banner/cluster

sudo cp -r news /var/www/html/news
sudo cp -r cluster /var/www/html/cluster

## Day 20: Configure Nginx + PHP-FPM Using Unix Sock

Prompt:

> The Nautilus application development team is planning to launch a new PHP-based application, which they want to deploy on Nautilus infra in Stratos DC. The development team had a meeting with the production support team and they have shared some requirements regarding the infrastructure. Below are the requirements they shared:  
> 
> a. Install nginx on app server 2 , configure it to use port 8093 and its document root should be /var/www/html.  
> b. Install php-fpm version 8.3 on app server 2, it must use the unix socket /var/run/php-fpm/default.sock (create the parent directories if don't exist).  
> c. Configure php-fpm and nginx to work together.  
> d. Once configured correctly, you can test the website using curl http://stapp02:8093/index.php command from jump host.  
>   
> NOTE: We have copied two files, index.php and info.php, under /var/www/html as part of the PHP-based application setup. Please do not modify these files.




ssh steve@stapp02

Install deps:
sudo dnf install nginx

https://www.php.net/downloads.php?usage=web&os=linux&osvariant=linux-redhat&version=8.3
```bash
$ sudo dnf install -y dnf-plugins-core
$ sudo dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-$(rpm -E %rhel).noarch.rpm
$ sudo dnf install -y https://rpms.remirepo.net/enterprise/remi-release-$(rpm -E %rhel).rpm
$ sudo dnf module reset php -y
$ sudo dnf module enable php:remi-8.3 -y
$ sudo dnf install -y php
```

Official English Documentation: http://nginx.org/en/docs/

sudo vi /etc/nginx/nginx.conf
```
    server {
        listen       8095;
        listen       [::]:8095;
        server_name  _;
        root         /var/www/html;
        [...]
```

sudo mkdir -p /var/run/php-fpm/
sudo vi /etc/php-fpm.d/www.conf
```
listen = /var/run/php-fpm/default.sock
```

sudo vi /etc/nginx/conf.d/php-fpm.conf 
```
upstream php-fpm {
        server unix:/var/run/php-fpm/default.sock;
}
```

sudo vi /etc/nginx/conf.d/www.conf
```
server {
    location ~ \.php$ {
        root /var/www/html;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php-fpm/www.sock;
        fastcgi_index index.php;
        include fastcgi.conf;
    }
}
```

sudo nginx -t
sudo systemctl restart nginx
sudo systemctl restart php-fpm

curl http://stapp02:8093/index.php