---
title: "KodeKloud's 100 Days of DevOps: Day 11 - 20"
date: 2025-12-31 # YYYY-MM-DD
lastMod: 2026-07-15
summary: "A walkthrough for days 11 through 20 of KodeKloud's 100 Days of DevOps challenges."
# draft: true
series: ["KodeKloud's 100 Days of DevOps"]
series_order: 2
categories: ["DevOps", "KodeKloud"]
tags: ["devops", "linux", "nginx", "mysql"]
---

## Introduction

Welcome back all to another post in the 100 Days of DevOps series! This set of problems has another round of Linux configuration and troubleshooting, interacting with various services - all key skills. You'll see Nginx, MySQL, networking, and more in this group; this set of problems is particularly heavy on troubleshooting, which I'm always a big fan of. If you missed it, make sure to check out [the first post in the series](../kk_100devops_01_10/index.md) - I think you'll enjoy it if you're into this stuff as much as I am!

You'll also likely see some notes and WIP chickenscratch as I work my way through writing these solutions up - just know that I'm keeping my notes close and will turn them into something much better sounding soon. With all that out of the way - on to Day 11!

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

> [!QUOTE]+ Problem Prompt
> Our monitoring tool has reported an issue in Stratos Datacenter. One of our app servers has an issue, as its Apache service is not reachable on port 8088 (which is the Apache port). The service itself could be down, the firewall could be at fault, or something else could be causing the issue.
>  
> Use tools like telnet, netstat, etc. to find and fix the issue. Also make sure Apache is reachable from the jump host without compromising any security settings.
>  
> Once fixed, you can test the same using command curl http://stapp01:8088 command from jump host.
>  
> Note: Please do not try to alter the existing index.html code, as it will lead to task failure.
{icon="circle-question"}

Okay, so based on the problem prompt we need to be ready to check out the problems bottom-up - we'll start with `ssh`ing over as `tony` and taking a look at the service for `httpd`.

```console
[tony@stapp01 ~]$ sudo systemctl status httpd.service
× httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; preset: disabled)
     Active: failed (Result: exit-code) since Tue 2026-07-14 14:00:23 UTC; 1min 7s ago
       Docs: man:httpd.service(8)
    Process: 13177 ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND (code=exited, status=1/FAILURE)
   Main PID: 13177 (code=exited, status=1/FAILURE)
     Status: "Reading configuration..."
        CPU: 34ms

Jul 14 14:00:23 stapp01 systemd[1]: Starting The Apache HTTP Server...
Jul 14 14:00:23 stapp01 httpd[13177]: AH00558: httpd: Could not reliably determine the server's fully qualified >
Jul 14 14:00:23 stapp01 httpd[13177]: (98)Address already in use: AH00072: make_sock: could not bind to address >
Jul 14 14:00:23 stapp01 httpd[13177]: (98)Address already in use: AH00072: make_sock: could not bind to address >
Jul 14 14:00:23 stapp01 httpd[13177]: no listening sockets available, shutting down
Jul 14 14:00:23 stapp01 httpd[13177]: AH00015: Unable to open logs
Jul 14 14:00:23 stapp01 systemd[1]: httpd.service: Main process exited, code=exited, status=1/FAILURE
Jul 14 14:00:23 stapp01 systemd[1]: httpd.service: Failed with result 'exit-code'.
Jul 14 14:00:23 stapp01 systemd[1]: Failed to start The Apache HTTP Server.
```

Based on that error, something else has the port that `httpd` needs to successfully start. In an enterprise situation, you'd likely investigate and need to see what's taking up the port, if another team deployed it, if there are negative knock-on effects of removing the service...you get the gist. In a lab like this, though, we only care about `httpd` getting up and running so we can just nuke the service from orbit once we have a valid process ID. Let's take a look at what's listening on the TCP side with `ss` and the `-p` flag to grab the PID:

```console
[tony@stapp01 ~]$ sudo ss -antp
State           Recv-Q           Send-Q                     Local Address:Port                      Peer Address:Port           Process                                                           
LISTEN          0                128                              0.0.0.0:8080                           0.0.0.0:*               users:(("ttyd",pid=60,fd=11))                                    
LISTEN          0                128                              0.0.0.0:22                             0.0.0.0:*               users:(("sshd",pid=1576,fd=3))                                   
LISTEN          0                10                             127.0.0.1:8088                           0.0.0.0:*               users:(("sendmail",pid=12753,fd=4))                              
ESTAB           0                0                          10.244.49.127:22                       10.244.97.158:42724           users:(("sshd",pid=36880,fd=4),("sshd",pid=36711,fd=4))          
LISTEN          0                128                                 [::]:22                                [::]:*               users:(("sshd",pid=1576,fd=4))
[tony@stapp01 ~]$ sudo kill 12753
```

Now that we've killed off the `sendmail` process, let's try restarting the `httpd` service and with any luck, we'll have a functioning web server. After the service comes up, we confirm it's listening on the correct port with `ss`:

```console
[tony@stapp01 ~]$ sudo systemctl restart httpd.service
[tony@stapp01 ~]$ sudo systemctl status httpd.service
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; preset: disabled)
     Active: active (running) since Tue 2026-07-14 14:02:35 UTC; 9s ago
       Docs: man:httpd.service(8)
   Main PID: 37462 (httpd)
     Status: "Total requests: 0; Idle/Busy workers 100/0;Requests/sec: 0; Bytes served/sec:   0 B/sec"
      Tasks: 177 (limit: 404712)
     Memory: 15.0M
        CPU: 64ms
     CGroup: /system.slice/httpd.service
             ├─37462 /usr/sbin/httpd -DFOREGROUND
             ├─37469 /usr/sbin/httpd -DFOREGROUND
             ├─37470 /usr/sbin/httpd -DFOREGROUND
             ├─37471 /usr/sbin/httpd -DFOREGROUND
             └─37472 /usr/sbin/httpd -DFOREGROUND

Jul 14 14:02:35 stapp01 systemd[1]: Starting The Apache HTTP Server...
Jul 14 14:02:35 stapp01 httpd[37462]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.244.49.127. Set the 'ServerName' directive globally to supp>
Jul 14 14:02:35 stapp01 httpd[37462]: Server configured, listening on: port 8088
Jul 14 14:02:35 stapp01 systemd[1]: Started The Apache HTTP Server.
[tony@stapp01 ~]$ sudo ss -antp
State     Recv-Q    Send-Q       Local Address:Port         Peer Address:Port     Process                                                                                                         
LISTEN    0         128                0.0.0.0:8080              0.0.0.0:*         users:(("ttyd",pid=60,fd=11))                                                                                  
LISTEN    0         128                0.0.0.0:22                0.0.0.0:*         users:(("sshd",pid=1576,fd=3))                                                                                 
ESTAB     0         0            10.244.49.127:22          10.244.97.158:42724     users:(("sshd",pid=36880,fd=4),("sshd",pid=36711,fd=4))                                                        
LISTEN    0         511                      *:8088                    *:*         users:(("httpd",pid=37472,fd=4),("httpd",pid=37471,fd=4),("httpd",pid=37470,fd=4),("httpd",pid=37462,fd=4))    
LISTEN    0         128                   [::]:22                   [::]:*         users:(("sshd",pid=1576,fd=4))
```

Perfect! We now have a functioning `httpd` instance. The next step is to see if we can hit `index.html` locally with `curl`; if it works, we can mostly rule out our web server setup being the culprit.

```
[tony@stapp01 ~]$ curl -I localhost:8088
HTTP/1.1 403 Forbidden
Date: Tue, 14 Jul 2026 14:03:37 GMT
Server: Apache/2.4.62 (CentOS Stream)
Last-Modified: Fri, 12 Dec 2025 14:14:29 GMT
ETag: "296919-645c1e12b9b40"
Accept-Ranges: bytes
Content-Length: 2713881
Content-Type: text/html; charset=UTF-8
```

A `403 Forbidden` response isn't a good sign - it could mean a few things. The easiest check, though, is to see if we even _have_ an `index.html` living at the document root. To do that, I checked the `httpd.conf` to see where it expects the file, and then a simple `ls -al` to see if the files are present:

```console
[tony@stapp01 ~]$ grep -i documentroot /etc/httpd/conf/httpd.conf 
# DocumentRoot: The directory out of which you will serve your
DocumentRoot "/var/www/html"
    # access content that does not live under the DocumentRoot.
[tony@stapp01 ~]$ ls -la /var/www/html
total 8
drwxr-xr-x 2 root root 4096 Jun  1 13:26 .
drwxr-xr-x 4 root root 4096 Jun 10 09:03 ..
```

Now that we've confirmed the file isn't where it's expected, we have to find it. The easiest way to find a file is, well, to use the aptly-named `find`. Once we've located a valid file, we can copy it over to `/var/www/html` to resolve the `403 Forbidden` error:
```console
[tony@stapp01 ~]$ find / -name "index.html" 2>/dev/null
/usr/share/doc/oniguruma/index.html
/usr/share/nginx/html/index.html
/usr/share/testpage/index.html
/usr/share/httpd/noindex/index.html
[tony@stapp01 ~]$ sudo cp /usr/share/testpage/index.html /var/www/html
[tony@stapp01 ~]$ ls -la /var/www/html
total 2664
drwxr-xr-x 1 root root    4096 Jul 14 14:04 .
drwxr-xr-x 1 root root    4096 Jun 10 09:03 ..
-rw-r--r-- 1 root root 2713881 Jul 14 14:04 index.html
[tony@stapp01 ~]$ curl -I localhost:8088
HTTP/1.1 200 OK
Date: Tue, 14 Jul 2026 14:05:01 GMT
Server: Apache/2.4.62 (CentOS Stream)
Last-Modified: Tue, 14 Jul 2026 14:04:52 GMT
ETag: "296919-65692b0d67019"
Accept-Ranges: bytes
Content-Length: 2713881
Content-Type: text/html; charset=UTF-8
```

If we test access from `thor@jump-box`, we'll still see that there's an error - `No route to host`. There's a couple of networking issues that could be present for that, but we'll start by identifying which firewall is present on the box (in this case, `iptables`) and then list out the rules that it has enabled:

```console
[tony@stapp01 ~]$ which firewall-cmd
/usr/bin/which: no firewall-cmd in (/home/tony/.local/bin:/home/tony/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin)
[tony@stapp01 ~]$ which iptables
/usr/sbin/iptables
[tony@stapp01 ~]$ sudo iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     all  --  anywhere             anywhere             state RELATED,ESTABLISHED
ACCEPT     icmp --  anywhere             anywhere            
ACCEPT     all  --  anywhere             anywhere            
ACCEPT     tcp  --  anywhere             anywhere             state NEW tcp dpt:ssh
REJECT     all  --  anywhere             anywhere             reject-with icmp-host-prohibited

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         
REJECT     all  --  anywhere             anywhere             reject-with icmp-host-prohibited

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

At first, you might think that there's nothing blocking - so what gives? `iptables` rules process top-to-bottom, meaning that if it doesn't match a rule, it continues down the list. Since we don't have an explicit `ACCEPT` rule set up for port 8088, it slides all the way to the `REJECT` all at the end of the `INPUT` chain, ultimately leading to any outside source being stopped from connecting to our web server.

To fix this, we need to add a rule to the top of the chain (or at least _above_ the `REJECT` rule).

```
[tony@stapp01 ~]$ sudo iptables -I INPUT 1 -i eth0 -p TCP --dport 8088 -j ACCEPT
[tony@stapp01 ~]$ sudo iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:radan-http
ACCEPT     all  --  anywhere             anywhere             state RELATED,ESTABLISHED
ACCEPT     icmp --  anywhere             anywhere            
ACCEPT     all  --  anywhere             anywhere            
ACCEPT     tcp  --  anywhere             anywhere             state NEW tcp dpt:ssh
REJECT     all  --  anywhere             anywhere             reject-with icmp-host-prohibited

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         
REJECT     all  --  anywhere             anywhere             reject-with icmp-host-prohibited

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

Now that we've added that rule, we test back from `thor@jump-box` to see if we can reach the web service from outside:

```console
[thor@jump-host ~]$ curl -I http://stapp01:8088
HTTP/1.1 200 OK
Date: Tue, 14 Jul 2026 14:09:41 GMT
Server: Apache/2.4.62 (CentOS Stream)
Last-Modified: Tue, 14 Jul 2026 14:04:52 GMT
ETag: "296919-65692b0d67019"
Accept-Ranges: bytes
Content-Length: 2713881
Content-Type: text/html; charset=UTF-8
```

And with that, our web service is ready to go for this task!

## Day 13: IPtables Installation and Configuration

> [!QUOTE]+ Problem Prompt
> We have one of our websites up and running on our Nautilus infrastructure in Stratos DC. Our security team has raised a concern that right now Apache’s port i.e 8084 is open for all since there is no firewall installed on these hosts. So we have decided to add some security layer for these hosts and after discussions and recommendations we have come up with the following requirements:
> 
> 1. Install iptables and all its dependencies on each app host.
> 2. Block incoming port 8084 on all apps for everyone except for LBR host.
> 3. Make sure the rules remain, even after system reboot.
{icon="circle-question"}

As with other tasks in the series, I'll show all of the examples with `stapp01` only, but you'll need to repeat all of the steps shown against all three of the app servers in order to complete the task successfully. First, let's check the host to see what we have installed so that we can know what to grab from `dnf`:

```
[tony@stapp01 ~]$ dnf list --installed | grep iptables
iptables-libs.x86_64                           1.8.10-11.el9                    @baseos       
iptables-services.noarch                       1.8.10-11.1.el9                  @epel
```

So we have the supporting libraries, but not the `iptables` binaries themselves. Let's install them and enable the service so that we can set up the firewall rules.

```console
[tony@stapp01 ~]$ sudo dnf install -y iptables
Last metadata expiration check: 0:15:24 ago on Tue Jul 14 14:32:04 2026.
Dependencies resolved.
=================================================================================================================
 Package                            Architecture         Version                        Repository          Size
=================================================================================================================
Installing:
 iptables-legacy                    x86_64               1.8.10-11.1.el9                epel                50 k
Installing dependencies:
 iptables-legacy-libs               x86_64               1.8.10-11.1.el9                epel                38 k

                        [...]

  Verifying        : iptables-legacy-libs-1.8.10-11.1.el9.x86_64                                             2/2 

Installed:
  iptables-legacy-1.8.10-11.1.el9.x86_64               iptables-legacy-libs-1.8.10-11.1.el9.x86_64              

Complete!
[tony@stapp01 ~]$ sudo systemctl enable --now iptables.service
Created symlink /etc/systemd/system/multi-user.target.wants/iptables.service → /usr/lib/systemd/system/iptables.service.
[tony@stapp01 ~]$ sudo systemctl status iptables.service
● iptables.service - IPv4 firewall with iptables
     Loaded: loaded (/usr/lib/systemd/system/iptables.service; enabled; preset: disabled)
     Active: active (exited) since Tue 2026-07-14 20:48:11 UTC; 10s ago
    Process: 46099 ExecStart=/usr/libexec/iptables/iptables.init start (code=exited, status=0/SUCCESS)
   Main PID: 46099 (code=exited, status=0/SUCCESS)
        CPU: 12ms

Jul 14 20:48:11 stapp01 systemd[1]: Starting IPv4 firewall with iptables...
Jul 14 20:48:11 stapp01 iptables.init[46099]: iptables: Applying firewall rules: [  OK  ]
Jul 14 20:48:11 stapp01 systemd[1]: Finished IPv4 firewall with iptables.
```

By installing and enabling the service, we're not _actually_ enabling anything yet. No traffic will be blocked, no services protected. We can verify this by listing rules with `iptables -L`:

```
[tony@stapp01 ~]$ sudo iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

And also by testing the connectivity from an outside source, in this case the jump box host:

```console
[thor@jump-host ~]$ curl -I http://stapp01:8084
HTTP/1.1 403 Forbidden
Date: Tue, 14 Jul 2026 20:31:08 GMT
Server: Apache/2.4.62 (CentOS Stream)
Last-Modified: Fri, 12 Dec 2025 14:14:29 GMT
ETag: "296919-645c1e12b9b40"
Accept-Ranges: bytes
Content-Length: 2713881
Content-Type: text/html; charset=UTF-8
```

The rule system is fairly straightforward for `iptables` rules are processed one at a time, from top to bottom, in the chain that matches the traffic pattern that applies. Our goal is to prevent traffic to port 8084 from all sources _except_ from the load balancer. The easiest way to do that is to put a "reject all" rule at the bottom of the ruleset and then allow the ports we _do_ want enabled.

This is the rough order of operations for setting up the rules:
- First, we have to add a rule allowing SSH - otherwise we won't be able to connect to the box once the `REJECT` rule gets added
- Next, we append a reject all rule to the bottom of the ruleset
- Finally, we add an allow for `stlb01` to reach the port in question

With that out of the way, let's add our rules and then list out our ruleset to confirm. Note that because the KodeKloud labs are spawned via Kubernetes calls, `stlb01` will translate automatically to `10-244-29-245.stlb01.3vvlmha5kdtyhwcz.svc.cluster.local`:

```console
[tony@stapp01 ~]$ sudo iptables -I INPUT -p TCP --dport 22 -j ACCEPT
[tony@stapp01 ~]$ sudo iptables -A INPUT -p TCP -j REJECT
[tony@stapp01 ~]$ sudo iptables -I INPUT -p TCP --dport 8084 -s stlb01 -j ACCEPT
[tony@stapp01 ~]$ sudo iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     tcp  --  10-244-29-245.stlb01.3vvlmha5kdtyhwcz.svc.cluster.local  anywhere             tcp dpt:rfe
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:ssh
REJECT     tcp  --  anywhere             anywhere             reject-with icmp-port-unreachable

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

With those rules in place, let's test them before making them permanent. We'll re-run the `curl` from earlier on the jump box again:

```console
thor@jump-host ~$ curl -I http://stapp01:8084
curl: (7) Failed to connect to stapp01 port 8084: Connection refused
```

And to confirm that the port is open to the load balancer, we test the same command from `stlb01`:

```console
[loki@stlb01 ~]$ curl -I http://stapp01:8084
HTTP/1.1 403 Forbidden
Date: Tue, 14 Jul 2026 20:35:30 GMT
Server: Apache/2.4.62 (CentOS Stream)
Last-Modified: Fri, 12 Dec 2025 14:14:29 GMT
ETag: "296919-645c1e12b9b40"
Accept-Ranges: bytes
Content-Length: 2713881
Content-Type: text/html; charset=UTF-8
```

Perfect! We have the rules enabled, connectivity from `stlb01`, and all that's left is to permanently save the rules so that they can be enabled from the get-go. To do that, we use the `iptables-save` command, saving the output to `/etc/sysconfig/iptables` since we're on a CentOS host:

```console
[tony@stapp01 ~]$ sudo sh -c '/sbin/iptables-save > /etc/sysconfig/iptables'
```

Now all that's left is to repeat the process for both `stapp02` and `stapp03`!

## Day 14: Linux Process Troubleshooting

> [!QUOTE]+ Problem Prompt
> The production support team of xFusionCorp Industries has deployed some of the latest monitoring tools to keep an eye on every service, application, etc. running on the systems. One of the monitoring systems reported about Apache service unavailability on one of the app servers in Stratos DC.
> 
> Identify the faulty app host and fix the issue. Make sure Apache service is up and running on all app hosts. They might not have hosted any code yet on these servers, so you don't need to worry if Apache isn't serving any pages. Just make sure the service is up and running. Also, make sure Apache is running on port 8085 on all app servers.
{icon="circle-question"}

From the problem prompt, we know that one or more hosts have _some kind of issue_. Since that's nebulous and we know the final desired state, i.e. Apache is up and running on port 8085, let's see if we can call the service on each of the hosts as-is:

```console
thor@jump-host ~$ curl -I http://stapp01:8085
curl: (7) Failed to connect to stapp01 port 8085: Connection refused
thor@jump-host ~$ curl -I http://stapp02:8085
HTTP/1.1 403 Forbidden
Date: Tue, 14 Jul 2026 21:30:28 GMT
Server: Apache/2.4.62 (CentOS Stream)
Last-Modified: Fri, 12 Dec 2025 14:14:29 GMT
ETag: "296919-645c1e12b9b40"
Accept-Ranges: bytes
Content-Length: 2713881
Content-Type: text/html; charset=UTF-8

thor@jump-host ~$ curl -I http://stapp03:8085
HTTP/1.1 403 Forbidden
Date: Tue, 14 Jul 2026 21:30:32 GMT
Server: Apache/2.4.62 (CentOS Stream)
Last-Modified: Fri, 12 Dec 2025 14:14:29 GMT
ETag: "296919-645c1e12b9b40"
Accept-Ranges: bytes
Content-Length: 2713881
Content-Type: text/html; charset=UTF-8
```

Right off the bat, we know that `stapp01` has issues - and it's likely the only one, given `stapp02` and `stapp03` return the same response to our `curl`. We'll SSH in to `stapp01` and start poking around. Let's start with the service status to see if that nets us any good troubleshooting info:

```console
[tony@stapp01 ~]$ sudo systemctl status httpd.service
× httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; preset: disabled)
     Active: failed (Result: exit-code) since Tue 2026-07-14 21:16:53 UTC; 14min ago
       Docs: man:httpd.service(8)
    Process: 20807 ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND (code=exited, status=1/FAILURE)
   Main PID: 20807 (code=exited, status=1/FAILURE)
     Status: "Reading configuration..."
        CPU: 34ms

Jul 14 21:16:53 stapp01 systemd[1]: Starting The Apache HTTP Server...
Jul 14 21:16:53 stapp01 httpd[20807]: AH00558: httpd: Could not reliably determine the server's fully qualified >
Jul 14 21:16:53 stapp01 httpd[20807]: (98)Address already in use: AH00072: make_sock: could not bind to address >
Jul 14 21:16:53 stapp01 httpd[20807]: (98)Address already in use: AH00072: make_sock: could not bind to address >
Jul 14 21:16:53 stapp01 httpd[20807]: no listening sockets available, shutting down
Jul 14 21:16:53 stapp01 httpd[20807]: AH00015: Unable to open logs
Jul 14 21:16:53 stapp01 systemd[1]: httpd.service: Main process exited, code=exited, status=1/FAILURE
Jul 14 21:16:53 stapp01 systemd[1]: httpd.service: Failed with result 'exit-code'.
Jul 14 21:16:53 stapp01 systemd[1]: Failed to start The Apache HTTP Server.
```

From that error output, we can see that the service can't get the port it's requesting - let's see if there's something else taking up the port using `ss` with the PID flag:

```console
[tony@stapp01 ~]$ sudo ss -antp
State         Recv-Q        Send-Q                  Local Address:Port                 Peer Address:Port         Process                                                                                                          
LISTEN        0             128                           0.0.0.0:8080                      0.0.0.0:*             users:(("ttyd",pid=60,fd=11))                                                                                   
LISTEN        0             128                           0.0.0.0:22                        0.0.0.0:*             users:(("sshd",pid=1466,fd=3))                                                                                  
LISTEN        0             10                          127.0.0.1:8085                      0.0.0.0:*             users:(("sendmail",pid=20159,fd=4))                                                                             
ESTAB         0             0                      10.244.234.248:22                   10.244.49.48:35050         users:(("sshd",pid=47384,fd=4),("sshd",pid=47258,fd=4))                                                         
LISTEN        0             128                              [::]:22                           [::]:*             users:(("sshd",pid=1466,fd=4))
```

So there's a `sendmail` instance running once again - as mentioned in Day 12, we'd normally take more care, but in this lab scenario we'll just kill the process that's taking the port and restart our Apache service:

```console
[tony@stapp01 ~]$ sudo kill 20159
[tony@stapp01 ~]$ sudo systemctl restart httpd.service
[tony@stapp01 ~]$ sudo systemctl status httpd.service
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; preset: disabled)
     Active: active (running) since Tue 2026-07-14 21:32:54 UTC; 6s ago
       Docs: man:httpd.service(8)
   Main PID: 48052 (httpd)
     Status: "Started, listening on: port 8085"
      Tasks: 177 (limit: 404712)
     Memory: 15.0M
        CPU: 61ms
     CGroup: /system.slice/httpd.service
             ├─48052 /usr/sbin/httpd -DFOREGROUND
             ├─48059 /usr/sbin/httpd -DFOREGROUND
             ├─48060 /usr/sbin/httpd -DFOREGROUND
             ├─48061 /usr/sbin/httpd -DFOREGROUND
             └─48062 /usr/sbin/httpd -DFOREGROUND

Jul 14 21:32:54 stapp01 systemd[1]: Starting The Apache HTTP Server...
Jul 14 21:32:54 stapp01 httpd[48052]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 10.244.234.248. Set the 'ServerName' directive globally to sup>
Jul 14 21:32:54 stapp01 httpd[48052]: Server configured, listening on: port 8085
Jul 14 21:32:54 stapp01 systemd[1]: Started The Apache HTTP Server.
```

Now that that's up, we need to check from the jump box. That's easily done with our `curl` from earlier:

```console
thor@jump-host ~$ curl -I http://stapp01:8085
HTTP/1.1 403 Forbidden
Date: Tue, 14 Jul 2026 21:33:21 GMT
Server: Apache/2.4.62 (CentOS Stream)
Last-Modified: Fri, 12 Dec 2025 14:14:29 GMT
ETag: "296919-645c1e12b9b40"
Accept-Ranges: bytes
Content-Length: 2713881
Content-Type: text/html; charset=UTF-8
```

With that, we're finished with this task - on to Day 15!

## Day 15: Setup SSL for Nginx

> [!QUOTE]+ Problem Prompt
> The system admins team of xFusionCorp Industries needs to deploy a new application on App Server 1 in Stratos Datacenter. They have some pre-requites to get ready that server for application deployment. Prepare the server as per requirements shared below:
> 
> 1. Install and configure nginx on App Server 1.
> 2. On App Server 1 there is a self signed SSL certificate and key present at location /tmp/nautilus.crt and /tmp/nautilus.key. Move them to some appropriate location and deploy the same in Nginx.
> 3. Create an index.html file with content Welcome! under Nginx document root.
> 4. For final testing try to access the App Server 1 link (via hostname) from jump host using curl command. For example: curl -Ik https://<app-server-name>/.
{icon="circle-question"}

This task is pretty straightforward - not too much in the way of thought process or variability in how to approach it. We'll start by installing Nginx with `dnf` and then enabling and starting the service with `systemctl`:

```console
[tony@stapp01 ~]$ sudo dnf install -y nginx
CentOS Stream 9 - BaseOS                     4.4 MB/s | 9.0 MB     00:02    
CentOS Stream 9 - AppStream                   13 MB/s |  28 MB     00:02    
CentOS Stream 9 - Extras packages             31 kB/s |  21 kB     00:00    
                  [...]
Complete!
[tony@stapp01 ~]$ sudo systemctl enable --now nginx.service
Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service → /usr/lib/systemd/system/nginx.service.
[tony@stapp01 ~]$ sudo systemctl status nginx.service
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset:>
     Active: active (running) since Wed 2026-07-15 00:38:24 UTC; 7s ago
    Process: 36164 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, >
    Process: 36172 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SU>
    Process: 36185 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 36192 (nginx)
                  [...]
```

Now that the service is up, we need to configure the site. If you're unfamiliar with how Nginx does things, all of the configuration files are housed under `/etc/nginx/`. The primary configuration file is `/etc/nginx/nginx.conf`, although occasionally you'll see setups that use drop-ins under `/etc/nginx/conf.d/*.conf`. It's nice for more complex setups, but for our use case with this lab, we'll stick to working with just the main configuration file.

Looking through `nginx.conf`, starting on line 56 of the file we'll see the SSL server section:

```nginx
                  [...]
# Settings for a TLS enabled server.
#
#    server {
#        listen       443 ssl http2;
#        listen       [::]:443 ssl http2;
#        server_name  _;
#        root         /usr/share/nginx/html;
#
#        ssl_certificate "/etc/pki/nginx/server.crt";
#        ssl_certificate_key "/etc/pki/nginx/private/server.key";
#        ssl_session_cache shared:SSL:1m;
#        ssl_session_timeout  10m;
#        ssl_ciphers PROFILE=SYSTEM;
#        ssl_prefer_server_ciphers on;
#
#        # Load configuration files for the default server block.
#        include /etc/nginx/default.d/*.conf;
#
#        error_page 404 /404.html;
#            location = /40x.html {
#        }
#
#        error_page 500 502 503 504 /50x.html;
#            location = /50x.html {
#        }
#    }
                  [...]
```

As you can see, the parts we're interested in are the ones that map to the problem prompt:
- The SSL certificate is stored at `/etc/pki/nginx/server.crt`
- The SSL private key is stored at `/etc/pki/nginx/private/server.key`
- The document root is at `/usr/share/nginx/html` - this is where we'll put the `index.html` file later

Now we just need to set up the files, uncomment the server block, and then restart the service.

First, we'll make the index page:
```
[tony@stapp01 ~]$ sudo sh -c 'echo "Welcome!" > /usr/share/nginx/html/index.html'
[tony@stapp01 ~]$ sudo ls -la /usr/share/nginx/html/index.html
lrwxrwxrwx 1 root root 25 May 27 10:16 /usr/share/nginx/html/index.html -> ../../testpage/index.html
[tony@stapp01 ~]$ sudo cat /usr/share/nginx/html/index.html
Welcome!
```
Now, we move the certs. Note that we had to create the `/etc/pki/nginx/private` directory, since it doesn't exist by default:
```console
[tony@stapp01 ~]$ sudo ls -al /etc/pki/nginx/private/
ls: cannot access '/etc/pki/nginx/private/': No such file or directory
[tony@stapp01 ~]$ sudo mkdir -p /etc/pki/nginx/private/
[tony@stapp01 ~]$ sudo cp /tmp/nautilus.crt /etc/pki/nginx/server.crt
[tony@stapp01 ~]$ sudo cp /tmp/nautilus.key /etc/pki/nginx/private/server.key
```

And finally, we need to modify `/etc/nginx/nginx.conf` and uncomment all of the lines in the SSL `server` block. Once we've done that, we can restart the service and have an SSL server running!

```console
[tony@stapp01 ~]$ # We uncomment all of the SSL server block in the
[tony@stapp01 ~]$ # nginx.conf file.
[tony@stapp01 ~]$ sudo vi /etc/nginx/nginx.conf
[tony@stapp01 ~]$ sudo systemctl restart nginx.service
```

Now we just need to check from the jump box to make sure that we have successfully deployed the SSL server. We can do so using the `-k` flag with `curl` to disable certificate checking, otherwise it won't accept the self-signed certs we used:

```console
thor@jump-host ~$ curl -Ik https://stapp01
HTTP/2 200 
server: nginx/1.20.1
date: Wed, 15 Jul 2026 00:46:20 GMT
content-type: text/html
content-length: 9
last-modified: Wed, 15 Jul 2026 00:41:09 GMT
etag: "6a56d725-9"
accept-ranges: bytes
```

Day 15 down!

## Day 16: Install and Configure Nginx as an LBR

> [!QUOTE]+ Problem Prompt
> TBD
{icon="circle-question"}

Placeholder.

## Day 17: Install and Configure PostgreSQL

> [!QUOTE]+ Problem Prompt
> TBD
{icon="circle-question"}

Placeholder.

## Day 18: Configure LAMP Server (OLD)

> [!QUOTE]+ Problem Prompt
> xFusionCorp Industries is planning to host a WordPress website on their infra in Stratos Datacenter. They have already done infrastructure configuration—for example, on the storage server they already have a shared directory /vaw/www/html that is mounted on each app host under /var/www/html directory. Please perform the following steps to accomplish the task:  
> 
> A. Install httpd, php and its dependencies on all app hosts.  
> B. Apache should serve on port 5004 within the apps.  
> C. Install/Configure MariaDB server on DB Server.  
> D. Create a database named kodekloud_db5 and create a database user named kodekloud_cap identified as password B4zNgHA7Ya. Further make sure this newly created user is able to perform all operation on the database you created.  
> E. Finally you should be able to access the website on LBR link, by clicking on the App button on the top bar. You should see a message like App is able to connect to the database using user kodekloud_cap  
{icon="circle-question"}

> [!WARNING]+ Problem Change
> It looks like at some point after I wrote this, KodeKloud retired the old problem and swapped it out for a new one. I'm leaving this in here so that you can see it - it's got some fun details scattered inside. It's not structured the same as my old ones and may not be fully complete, but I can't go back and re-do or check it now that they've replaced the problem set!
{icon="triangle-exclamation"}

Okay, this shouldn't be too bad. Let's take stock of what's already on the servers and see what we need to get this running. I'm starting on `stapp01` and will repeat the steps where needed on the other app servers. First up, what packages are installed? We can check the installed packages with `dnf` and grep to filter:

```console
[tony@stapp01 ~]$ dnf list --installed | grep httpd
[tony@stapp01 ~]$ dnf list --installed | grep php
```
It looks like we'll need to install both the server and the language. To do that, we run the following:

```console
[tony@stapp01 ~]$ sudo dnf install -y httpd php
```

Next, let's check out the structure of the app we're deploying:

```console
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

```console
[tony@stapp01 ~]$ sudo dnf install php-mysqlnd
```

Next up - we need to set Apache to bind to the correct port so that way the load balancer can reach the server. The default config for `httpd` on Fedora lives at `/etc/httpd/conf/httpd.conf`. You can edit it using Vi, Vim, Nano, or any other text or stream editors you know - I'm using `sed` for repeatability and consistency across the other servers.

```console
[tony@stapp01 ~]$ sudo sed -i 's/Listen 80/Listen 5004/g' /etc/httpd/conf/httpd.conf
[tony@stapp01 ~]$ sudo systemctl restart httpd
```

After restarting the service, you can verify the changes are in effect by running `systemctl status httpd` or checking the listening ports by using `sudo ss -antp | grep 5004` and making sure everything appears correct. With that, we can move over to the database portion of the setup.

The database first needs installation - confirm that MariaDB isn't already present using `dnf`, and then run the following if it's missing:

```console
[tony@stapp01 ~]$ sudo dnf install mariadb-server
[tony@stapp01 ~]$ sudo systemctl enable --now mariadb
```

Once it's installed and the service is available, we can start configuring the defaults. Run through the `mysql_secure_installation` script, choosing options as you go along - most of the defaults are fine - and then edit the bind-address in our MariaDB configuration to listen on `0.0.0.0` to accept remote connections.

```console
[tony@stapp01 ~]$ sudo mysql_secure_installation
[tony@stapp01 ~]$ # We'll uncomment the bind-address line with vi
[tony@stapp01 ~]$ sudo vi /etc/my.cnf.d/mariadb-server.cnf 
```

Finally, we need to set up the database user so that our apps can have access to run queries, update tables, etc. The demo site doesn't do any of that, but will just check if the connection is live in the `mysqli_connect` call from the code at the beginning of this section. I chose to limit the IP range to 172.16.X, since that's what range the containers were running with for both IP addresses:

```console
[tony@stapp01 ~]$ sudo mysql -u root -p
Enter password:
MariaDB [(none)]> CREATE DATABASE kodekloud_db5;
MariaDB [(none)]> CREATE USER 'kodekloud_cap'@'172.16.%' IDENTIFIED BY 'B4zNgHA7Ya';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON kodekloud_db5.* TO 'kodekloud_cap'@'172.16.%';
MariaDB [(none)]> FLUSH PRIVILEGES;
```

## Day 18: Install and Configure DB Server
> [!QUOTE]+ Problem Prompt
> We need to setup a database server on Nautilus DB Server in Stratos Datacenter. Please perform the below given steps on DB Server:
> 
> A. Install/Configure MariaDB server.  
> B. Create a database named kodekloud_db6.  
> C. Create a user called kodekloud_sam and set its password to BruCStnMT5.  
> D. Grant full permissions to user kodekloud_sam on database kodekloud_db6.  
{icon="circle-question"}

Placeholder.

## Day 19: Install and Configure Web Application

> [!QUOTE]+ Problem Prompt
> xFusionCorp Industries is planning to host two static websites on their infra in Stratos Datacenter. The development of these websites is still in-progress, but we want to get the servers ready. Please perform the following steps to accomplish the task:
>  
> A. Install httpd package and dependencies on app server 3.  
> B. Apache should serve on port 8083.  
> C. There are two website's backups /home/thor/news and /home/thor/cluster on jump_host. Set them up on Apache in a way that news should work on the link http://localhost:8083/news/ and cluster should work on link http://localhost:8083/cluster/ on the mentioned app server.  
> D. Once configured you should be able to access the website using curl command on the respective app server, i.e curl http://localhost:8083/news/ and curl http://localhost:8083/cluster/  
{icon="circle-question"}

ssh banner@stapp03

sudo sed -i "s/Listen 80/Listen 8083/g" /etc/httpd/conf/httpd.conf
sudo systemctl restart httpd

scp -r news banner@stapp03:/home/banner/news
scp -r cluster banner@stapp03:/home/banner/cluster

sudo cp -r news /var/www/html/news
sudo cp -r cluster /var/www/html/cluster

## Day 20: Configure Nginx + PHP-FPM Using Unix Sock

> [!QUOTE]+ Problem Prompt
> The Nautilus application development team is planning to launch a new PHP-based application, which they want to deploy on Nautilus infra in Stratos DC. The development team had a meeting with the production support team and they have shared some requirements regarding the infrastructure. Below are the requirements they shared:  
> 
> A. Install nginx on app server 2 , configure it to use port 8093 and its document root should be /var/www/html.  
> B. Install php-fpm version 8.3 on app server 2, it must use the unix socket /var/run/php-fpm/default.sock (create the parent directories if don't exist).  
> C. Configure php-fpm and nginx to work together.  
> D. Once configured correctly, you can test the website using curl http://stapp02:8093/index.php command from jump host.  
>   
> NOTE: We have copied two files, index.php and info.php, under /var/www/html as part of the PHP-based application setup. Please do not modify these files.
{icon="circle-question"}


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