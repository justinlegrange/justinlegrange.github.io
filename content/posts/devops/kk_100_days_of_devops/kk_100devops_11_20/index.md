---
title: "KodeKloud's 100 Days of DevOps: Day 11 - 20"
date: 2025-12-31 # YYYY-MM-DD
lastMod: 2026-06-24
summary: "A walkthrough for days 11 through 20 of KodeKloud's 100 Days of DevOps challenges."
draft: true
series: ["KodeKloud's 100 Days of DevOps"]
series_order: 2
categories: ["DevOps", "KodeKloud"]
tags: ["devops", "linux", "kodekloud"]
---

## Introduction

Welcome back all to another post in the 100 Days of DevOps series! 

> [!IMPORTANT]+ Spoiler alert!
> In case you're squeamish about this sort of thing, there are a bunch of spoilers ahead - proceed at your own (self-learning) risk. I'll be diving into the nitty-gritty behind solutions where I can, so hopefully you'll be able to learn a thing or two.  
> 
> It's also worth noting that if you're working alongside me, you'll see different users, IP addresses, passwords, or even completely different solutions occasionally - they rotate these with each challenge spawn on most challenges.
{icon="circle-info"}

## Day 11: Install and Configure Tomcat Server

> [!QUOTE]+ Problem Prompt
> The Nautilus application development team recently finished the beta version of one of their Java-based applications, which they are planning to deploy on one of the app servers in Stratos DC. After an internal team meeting, they have decided to use the tomcat application server. Based on the requirements mentioned below complete the task:  
>  
> A. Install tomcat server on App Server 1.  
> B. Configure it to run on port 6300.  
> C. There is a ROOT.war file on Jump host at location /tmp.  
>  
> Deploy it on this tomcat server and make sure the webpage works directly on base URL i.e curl http://stapp01:6300
{icon="circle-question"}

For those unfamiliar, Tomcat is an open-source Apache web server based on Java. We can use it to deploy Java servlets for use on the web. It's got quite a bit of features and documentation, but to get off the ground we can take a look at the [`RUNNING.txt` file](https://tomcat.apache.org/tomcat-11.0-doc/RUNNING.txt) that comes bundled with the server. They host it at that link for convenience, but here are the important overarching steps:
1. Download and Install a Java SE Runtime Environment (JRE)
2. Download and Install Apache Tomcat
3. Configure Environment Variables
4. Start Up Tomcat

In addition to those steps, I'm also going to set up a separate `tomcat` user and group so that we're running the eventual SystemD service under its own dedicated, non-root user. It's good for system security as well as segmentation of data and privileges, and is generally considered best practice for production deployments. If you read through [Day 1 of the series](../kk_100devops_01_10/index.md), this should all look really familiar:

```console
[tony@stapp01 ~]$ sudo groupadd tomcat
[tony@stapp01 ~]$ sudo useradd -M -s /sbin/nologin -g tomcat -d /opt/tomcat tomcat
[tony@stapp01 ~]$ grep tomcat /etc/passwd
tomcat:x:1001:1001::/opt/tomcat:/sbin/nologin
```

Now that the user is in place, let's check whether or not we have any version(s) of the Java JDK installed on the server. I'm choosing to do this based on experience, knowing that Tomcat is a project based on Java. We can easily do this with `dnf`:

```console
[tony@stapp01 ~]$ dnf list --installed | grep java
java-11-openjdk.x86_64                         1:11.0.20.1.1-2.el9              @appstream    
java-11-openjdk-headless.x86_64                1:11.0.20.1.1-2.el9              @appstream    
javapackages-filesystem.noarch                 6.0.0-4.el9                      @appstream    
tzdata-java.noarch                             2024a-2.el9                      @appstream
[tony@stapp01 ~]$ java -version
openjdk version "11.0.20.1" 2023-08-24 LTS
OpenJDK Runtime Environment (Red_Hat-11.0.20.1.1-2) (build 11.0.20.1+1-LTS)
OpenJDK 64-Bit Server VM (Red_Hat-11.0.20.1.1-2) (build 11.0.20.1+1-LTS, mixed mode, sharing)
```

Another way to tell you'll need one of the OpenJDK verions is by taking a look at the ["Which Version?" page](https://tomcat.apache.org/whichversion.html) from the Tomcat site; you'll see that we need Java 17 or above in order to run Tomcat 11. Currently, we show Java 11 locally - if we don't change this out, we'll get an error in `$CATALINA_HOME/logs/catalina.out` when we try to start the service later:

```console
[tony@stapp01 tomcat]$ sudo bin/startup.sh
Using CATALINA_BASE:   /opt/tomcat
Using CATALINA_HOME:   /opt/tomcat
Using CATALINA_TMPDIR: /opt/tomcat/temp
Using JRE_HOME:        /
Using CLASSPATH:       /opt/tomcat/bin/bootstrap.jar:/opt/tomcat/bin/tomcat-juli.jar
Using CATALINA_OPTS:   
Tomcat started.
[tony@stapp01 tomcat]$ sudo cat logs/catalina.out
Unrecognized option: --enable-native-access=ALL-UNNAMED
Error: Could not create the Java Virtual Machine.
Error: A fatal exception has occurred. Program will exit.
```

Removing the old Java 11 JDK and installing a new version of Java is very straightforward - DNF has lots of versions above Java 17 available to choose from. Pick whichever your preferred version is and install it like so (I'm installing `java-25-openjdk`):

```console
[tony@stapp01 tomcat]$ sudo dnf remove -y java-11-openjdk
Dependencies resolved.
=============================================================================
 Package                   Arch    Version                 Repository   Size
=============================================================================
Removing:
 java-11-openjdk           x86_64  1:11.0.20.1.1-2.el9     @appstream  1.0 M
Removing unused dependencies:
 copy-jdk-configs          noarch  4.0-3.el9               @appstream   19 k
 java-11-openjdk-headless  x86_64  1:11.0.20.1.1-2.el9     @appstream  169 M
 lua                       x86_64  5.4.4-4.el9             @appstream  593 k
 lua-posix                 x86_64  35.0-8.el9              @appstream  614 k

Transaction Summary
[...]

[tony@stapp01 tomcat]$ sudo dnf install -y java-25-openjdk
Last metadata expiration check: 0:45:52 ago on Sun Jul 12 23:06:32 2026.
Dependencies resolved.
=============================================================================
 Package                        Arch   Version               Repo       Size
=============================================================================
Installing:
 java-25-openjdk                x86_64 1:25.0.3.0.9-1.el9    appstream 385 k
Upgrading:
 glibc                          x86_64 2.34-273.el9          baseos    2.0 M
 glibc-common                   x86_64 2.34-273.el9          baseos    308 k
 glibc-devel                    x86_64 2.34-273.el9          appstream  39 k
 glibc-headers                  x86_64 2.34-273.el9          appstream 547 k
 glibc-minimal-langpack         x86_64 2.34-273.el9          baseos     23 k
 tzdata-java                    noarch 2026b-1.el9           appstream 223 k
Installing dependencies:
 java-25-openjdk-crypto-adapter x86_64 1:25.0.3.0.9-1.el9    appstream  46 k
 java-25-openjdk-headless       x86_64 1:25.0.3.0.9-1.el9    appstream  59 M
Installing weak dependencies:
 glibc-langpack-en              x86_64 2.34-273.el9          baseos    662 k

Transaction Summary
[...]
```

Perfect, we're now set up with a Java that will function with Tomcat 11. Now we need to download the `tomcat` tarball and extract it somewhere. I set up a directory at `/opt/tomcat` and extract it there:

```console
[tony@stapp01 ~]$ wget https://downloads.apache.org/tomcat/tomcat-11/v11.0.24/bin/apache-tomcat-11.0.24.tar.gz
--2026-07-12 20:03:48--  https://downloads.apache.org/tomcat/tomcat-11/v11.0.24/bin/apache-tomcat-11.0.24.tar.gz
Resolving downloads.apache.org (downloads.apache.org)... 95.216.224.44, 88.99.208.237, 2a01:4f9:2b:1cc2::2, ...
Connecting to downloads.apache.org (downloads.apache.org)|95.216.224.44|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 14443350 (14M) [application/x-gzip]
Saving to: ‘apache-tomcat-11.0.24.tar.gz’

apache-tomcat-11.0. 100%[================>]  13.77M  --.-KB/s    in 0.1s    

2026-07-12 20:03:49 (94.8 MB/s) - ‘apache-tomcat-11.0.24.tar.gz’ saved [14443350/14443350]

[tony@stapp01 ~]$ sudo mkdir /opt/tomcat
[tony@stapp01 ~]$ sudo tar xvf apache-tomcat-11.0.24.tar.gz -C /opt/tomcat --strip-components=1
apache-tomcat-11.0.24/conf/
apache-tomcat-11.0.24/conf/catalina.properties
apache-tomcat-11.0.24/conf/context.xml

        [...]

apache-tomcat-11.0.24/bin/tool-wrapper.sh
apache-tomcat-11.0.24/bin/version.sh

[tony@stapp01 ~]$ ls -la /opt/tomcat
total 172
drwxr-xr-x 9 root root  4096 Jul 12 20:18 .
drwxr-xr-x 1 root root  4096 Jul 12 20:17 ..
-rw-r----- 1 root root 26826 Jul  3 07:01 BUILDING.txt
-rw-r----- 1 root root  8626 Jul  3 07:01 CONTRIBUTING.md
-rw-r----- 1 root root 60517 Jul  3 07:01 LICENSE
-rw-r----- 1 root root  2333 Jul  3 07:01 NOTICE
-rw-r----- 1 root root  3224 Jul  3 07:01 README.md
-rw-r----- 1 root root  6470 Jul  3 07:01 RELEASE-NOTES
-rw-r----- 1 root root 16114 Jul  3 07:01 RUNNING.txt
drwxr-x--- 2 root root  4096 Jul 12 20:18 bin
drwx------ 2 root root  4096 Jul  3 07:01 conf
drwxr-x--- 2 root root  4096 Jul 12 20:18 lib
drwxr-x--- 2 root root  4096 Jul  3 07:01 logs
drwxr-x--- 2 root root  4096 Jul 12 20:18 temp
drwxr-x--- 7 root root  4096 Jul  3 07:01 webapps
drwxr-x--- 2 root root  4096 Jul  3 07:01 work
```

Now that it's extracted, we can move the `ROOT.war` servlet over to the `webapps` directory so that we can reach it over the web. We need to move it from the jumpbox to `stapp01` first, then to `/opt/tomcat/webapps`. Note the change in terminal - you can either use two terminal tabs or disconnect and reconnect after `scp`'ing the file:
```
[thor@jump-host ~]$ scp /tmp/ROOT.war tony@stapp01:/home/tony/ROOT.war
tony@stapp01's password: 
ROOT.war                                   100% 4529    10.0MB/s   00:00
[tony@stapp01 tomcat]$ sudo cp ~/ROOT.war /opt/tomcat/webapps/
```

Lastly, we'll need to configure the server to serve over port 6300 per the project instructions. This can be done by modifying the file `conf/server.xml` and changing the port on the `Connector` that's present. By default, you'll see it as `<Connector port="8080" [...]>`. After you modify the file, this is how it should look:

```console
[tony@stapp01 tomcat]$ sudo cat /opt/tomcat/conf/server.xml | grep -A2 6300
<Connector port="6300" protocol="HTTP/1.1"
            connectionTimeout="20000"
            redirectPort="8443" />
```

Finally, we need to give permissions to the Tomcat user so that it can start and stop the service as needed, as well as writing out the miscellaneous files it needs. The easiest way is to just `chown` all of the files and folders under `/opt/tomcat` to the `tomcat` user:

```console
[tony@stapp01 ~]$ sudo chown -R tomcat:tomcat /opt/tomcat
```

Now that we've done all of the prerequisite setup, we can finally move to getting the service up and running. 
To ensure the server runs consistently, we can create a SystemD service file at `/etc/systemd/system/tomcat.service`. Use whatever editor you like, but this is how it needs to look once it's all finished:

```systemd
[Unit]
Description=Tomcat Server
After=syslog.target network.target

[Service]
Type=forking
User=tomcat
Group=tomcat

Environment=JAVA_HOME=/usr/lib/jvm/jre
Environment='JAVA_OPTS=-Djava.awt.headless=true'
Environment=CATALINA_HOME=/opt/tomcat
Environment=CATALINA_BASE=/opt/tomcat
Environment=CATALINA_PID=/opt/tomcat/temp/tomcat.pid
Environment='CATALINA_OPTS=-Xms512M -Xmx1024M'
ExecStart=/opt/tomcat/bin/catalina.sh start
ExecStop=/opt/tomcat/bin/catalina.sh stop

[Install]
WantedBy=multi-user.target
```

Once that is saved, we can finally start the service. First we have to run a `daemon-reload` so that `systemctl` picks up the new service file, and then we enable and run the service. If all goes to plan, we'll have a service that shows as `active (running)`:

```console
[tony@stapp01 tomcat]$ sudo systemctl daemon-reload
[tony@stapp01 tomcat]$ sudo systemctl enable tomcat
Created symlink /etc/systemd/system/multi-user.target.wants/tomcat.service → /etc/systemd/system/tomcat.service.
[tony@stapp01 tomcat]$ sudo systemctl start tomcat
[tony@stapp01 tomcat]$ sudo systemctl status tomcat.service
● tomcat.service - Tomcat Server
     Loaded: loaded (/etc/systemd/system/tomcat.service; enabled; preset: di>
     Active: active (running) since Sun 2026-07-12 23:55:32 UTC; 6s ago
    Process: 48935 ExecStart=/opt/tomcat/bin/catalina.sh start (code=exited,>
   Main PID: 48961 (java)
      Tasks: 31 (limit: 404712)
     Memory: 122.3M
        CPU: 1.861s
     CGroup: /system.slice/tomcat.service
             └─48961 /usr/lib/jvm/jre/bin/java -Djava.util.logging.config.fi>

Jul 12 23:55:32 stapp01 systemd[1]: Starting Tomcat Server...
Jul 12 23:55:32 stapp01 catalina.sh[48935]: Existing PID file found during s>
Jul 12 23:55:32 stapp01 catalina.sh[48935]: Removing/clearing stale PID file.
Jul 12 23:55:32 stapp01 catalina.sh[48935]: Tomcat started.
Jul 12 23:55:32 stapp01 systemd[1]: Started Tomcat Server.
```

Last step - we can quickly verify that our server is running by checking with `curl`:

```console
[tony@stapp01 tomcat]$ curl http://localhost:6300
<!DOCTYPE html>
<!--
To change this license header, choose License Headers in Project Properties.
To change this template file, choose Tools | Templates
and open the template in the editor.
-->
<html>
    <head>
        <title>SampleWebApp</title>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
    </head>
    <body>
        <h2>Welcome to xFusionCorp Industries!</h2>
        <br>
    
    </body>
</html>
```

## Day 12: Linux Networking Services

Placeholder.

## Day 13: IPtables Installation and Configuration

Placeholder.

## Day 14: Linux Process Troubleshooting

Placeholder.

## Day 15: Setup SSL for Nginx

Placeholder.

## Day 16: Install and Configure Nginx as an LBR

Placeholder.

## Day 17: Install and Configure PostgreSQL

Placeholder.

## Day 18: Configure LAMP Server

> [!QUOTE] Problem Prompt
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

> [!QUOTE] Problem Prompt
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

> [!QUOTE] Problem Prompt
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