---
title: "KodeKloud's 100 Days of DevOps: Days 1 - 10"
date: 2025-12-16 # YYYY-MM-DD
lastmod: 2026-06-05
description: "A walkthrough for days 1 through 10 of KodeKloud's 100 Days of DevOps challenges."
summary: "A walkthrough for days 1 through 10 of KodeKloud's 100 Days of DevOps challenges."
# weight: 1
# aliases: ["/first"]
draft: true
series: ["KodeKloud's 100 Days of DevOps"]
categories: ["DevOps", "KodeKloud", "Sysadmin"]
tags: ["devops", "linux", "kodekloud", "bash", "sysadmin", "cron"]
showToc: true
TocOpen: false
hidemeta: false
# disableHLJS: false # to disable highlightjs
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

## Intro

Welcome to my series of blog posts centered around the KodeKloud "100 Days of DevOps" challenge, [hosted here](https://kodekloud.com/100-days-of-devops)! In case you haven't looked at it before, KK's 100 Days of DevOps is a fantastic challenge series where they give a lot of common server administration/DevOps specific tasks using their KodeKloud Engineer platform. 

You'll see things like Linux administration, Git configuration, Docker changes, and Kubernetes troubleshooting through the lens of an administrator doing the work. I'm a bit late to the party (it was released a good while back!), but I'm loving every bit of this - it's extremely satisfying getting your hands dirty and knocking down task-over-task.

> [!IMPORTANT]+ Spoiler alert!
> In case you're squeamish about this sort of thing, there are a bunch of spoilers ahead - proceed at your own (self-learning) risk. I'll be diving into the nitty-gritty behind solutions where I can, so hopefully you'll be able to learn a thing or two. It's also worth noting that if you're working alongside me, you'll see different users, IP addresses, or passwords occasionally - they rotate these with each challenge spawn on most challenges.
{icon="circle-info"}

## Day 1: User with Non-Interactive Shell
> [!QUOTE]+ Problem Prompt
> To accommodate the backup agent tool's specifications, the system admin team at xFusionCorp Industries requires the creation of a user with a non-interactive shell. Here's your task:  
> 
> Create a user named mark with a non-interactive shell on App Server 3.
{icon="circle-question"}

First things first - on Linux, what is a non-interactive shell, and how do we add a user to it? There are 4 types of shells in the Linux world that I could find:
- **Interactive login** - This is the type of shell you get when you SSH into a computer, or pop open the local TTY before your window manager/display environment loads
- **Interactive, non-login** - An example of this shell type is opening a new terminal in something like Konsole or Terminal
- **Non-interactive, login** - This one is new to me, but the basic gist is piping input into a login shell somehow, i.e. running a command like `echo hello | ssh <server IP>`. I learned it researching for this question and stumbling upon [this StackOverflow answer](https://askubuntu.com/questions/879364/differentiate-interactive-login-and-non-interactive-non-login-shell)
- **Non-interactive, non-login** - This shell type would be for scripts dropping into subshell

So for the task, we're going to create a service account effectively - the easiest way to do that is to set the shell to use the `/sbin/nologin` binary. We'll accomplish that with the built in `adduser` utility ([man page here, for the curious](https://linux.die.net/man/8/adduser)).

Why do we do this? From the `/sbin/nologin --help` command, the utility returns a polite message to the user if anyone tries to log in to the account:

```
/sbin/nologin --help

Usage:
 nologin [options]

Politely refuse a login.

Options:
 -c, --command <command>  does nothing (for compatibility with su -c)
 -h, --help               display this help
 -V, --version            display version

For more details see nologin(8).
```

So all that's left is to create the user `mark` on the system:

```console
[thor@jumphost ~]$  ssh banner@stapp03
[banner@stapp03 ~]$ sudo adduser -s /sbin/nologin -M mark
```

And the user is created! You can verify with a ton of different methods, but there's always the quick check in `/etc/passwd`:
```console
[banner@stapp03 ~]$ cat /etc/passwd | grep mark
mark:x:1002:1002::/home/mark:/sbin/nologin
```

## Day 2: Temp User with Expiry
> [!QUOTE]+ Problem Prompt
> As part of the temporary assignment to the Nautilus project, a developer
> named anita requires access for a limited duration. To ensure smooth
> access management, a temporary user account with an expiry date is needed.
> Here's what you need to do:
>  
> Create a user named anita on App Server 1 in Stratos Datacenter. Set the
> expiry date to 2024-04-15, ensuring the user is created in lowercase
> as per standard protocol.
{icon="circle-question"}

This one will be extremely similar to the previous day's task - we just need to add a user using the `adduser` utility. [Looking through the man page](https://linux.die.net/man/8/adduser), we see the `-e` option lets us specify an expiration date in `YYYY-MM-DD` format:

```console
[thor@jumphost ~]$ ssh tony@stapp01
[tony@stapp01 ~]$ sudo adduser -e 2024-04-15 anita
```

And we're finished! Let's verify the account was created correctly in two different ways - the quick check using `/etc/passwd`, and verifying by using the `chage` utility ([man page for the curious](https://linux.die.net/man/1/chage)):

```console
[tony@stapp01 ~]$ cat /etc/passwd | grep anita
anita:x:1002:1002::/home/anita:/bin/bash

[tony@stapp01 ~]$ sudo chage -l anita
Last password change                                    : Dec 16, 2025
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : Apr 15, 2024
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99999
Number of days of warning before password expires       : 7
```

## Day 3: Secure Root SSH Access
> [!QUOTE]+ Problem Prompt
> Following security audits, the xFusionCorp Industries security team has rolled
> out new protocols, including the restriction of direct root SSH login.  
>   
> Your task is to disable direct SSH root login on all app servers within the
> Stratos Datacenter.
{icon="circle-question"}

This one is relatively straightforward if you've done any sort of Linux administration before. For SSH, most of your configuration settings live in `/etc/ssh/sshd_config`. The particular attribute we're looking for is `PermitRootLogin` which, unsurprisingly, enables or disables logging in with the root account via SSH. They want us to run this against all of the web app servers - so we'll SSH in to `stapp01` to start the editing.

You can edit the line in whatever text editor/stream editor you fancy, in this case I'm running the change through `sed` (don't forget to restart the service!):

```console
[tony@stapp01 ~]$ sudo sed -i "s/PermitRootLogin yes/PermitRootLogin no/g" /etc/ssh/sshd_config
[tony@stapp01 ~]$ sudo systemctl restart sshd
```

To verify changes, we can try to SSH with the root account and be denied. Repeat the process for the other servers and you should be set!


## Day 4: Script Execution Permissions
> [!QUOTE]+ Problem Prompt
> In a bid to automate backup processes, the xFusionCorp Industries sysadmin team has developed a new bash script named xfusioncorp.sh. While the script has been distributed to all necessary servers, it lacks executable permissions on App Server 1 within the Stratos Datacenter.
>
> Your task is to grant executable permissions to the /tmp/xfusioncorp.sh script on App Server 1. Additionally, ensure that all users have the capability to execute it.
{icon="circle-question"}

This one seems pretty straightforward - we just need to set appropriate file permissions using `chmod`. Let's start off by setting up an SSH session as `tony@stapp01` and checking the permissions that already exist on the file:
```console
[tony@stapp01 ~]$ ls -la /tmp/xfusioncorp.sh
---------- 1 root root 40 Jun  6 02:26 /tmp/xfusioncorp.sh
```

So how exactly do we read that? The easiest way is to look at regular files and go from there - normally, the permissions should look something like the top two lines on the `stapp01's /etc/passwd`:
```console
[tony@stapp01 ~]$ ls -la /etc/passwd
-rw-r--r-- 1 root root 1126 Jun  6 02:02 /etc/passwd
|[-][-][-]  [---] [--]
| |  |  |     |     |
| |  |  |     |     +----------------> Group
| |  |  |     +----------------------> Owner
| |  |  |
| |  |  +----------------------------> Others Permissions
| |  +-------------------------------> Group Permissions
| +----------------------------------> Owner Permissions
+------------------------------------> File Type (directory, file, special types)
```

So in each position where there's a dash in the "Permissions" groups, you're likely going to see one of three main letters - `r` for read, `x` for execute, and `w` representing write permissions. If there's a dash, that means that particular permission isn't set and thus that group *doesn't* have access to that action. In the case of `/etc/passwd`, it reads as everyone (owner, group, others) having `read (r)` permissions, ONLY the owner (root) having `write (w)` permissions, and nobody having `execute (x)` permissions.

Okay, so on the `/tmp/xfusioncorp.sh` script we don't have any permissions for *anything*. In case you're not familiar, in order to execute scripts on Linux systems we typically need to set both the `execute` AND the `read` bits (more on that later). Doing that is straightforward with `chmod` and the symbolic notations. We can add read `(a+r)` and execute `(a+x)` to everyone with the following command:

```console
[tony@stapp01 ~]$ sudo chmod a+x,a+r /tmp/xfusioncorp.sh
[tony@stapp01 ~]$ ls -la /tmp/xfusioncorp.sh
-r-xr-xr-x 1 root root 40 Jun  6 02:26 /tmp/xfusioncorp.sh
```

And with that, we can finally verify that we can read and execute the file:

```console
[tony@stapp01 ~]$ cat /tmp/xfusioncorp.sh
#!/bin/bash

echo "Welcome To KodeKloud"
[tony@stapp01 ~]$ bash /tmp/xfusioncorp.sh 
Welcome To KodeKloud
```

Verifying that others can read the file is a bit trickier - if you try something like `su - steve`, you'll be hit with an `su` error: `su: user steve does not exist or the user entry does not contain all the required fields`. Checking `/etc/passwd` confirms that yes, you really are the only account on the box - likely just due to how KodeKloud provisions these on-demand. We'll just set up our own hack-y second user, change to them, and then execute the file anyways!

```console
[tony@stapp01 ~]$ sudo adduser -M extrauser
[tony@stapp01 ~]$ sudo su - extrauser
Last login: Sat Jun  6 03:06:12 UTC 2026 on pts/0
su: warning: cannot change directory to /home/extrauser: No such file or directory
[extrauser@stapp01 tony]$ whoami
extrauser
[extrauser@stapp01 tony]$ id
uid=1001(extrauser) gid=1001(extrauser) groups=1001(extrauser)
[extrauser@stapp01 tony]$ cat /tmp/xfusioncorp.sh
#!/bin/bash

echo "Welcome To KodeKloud"
[extrauser@stapp01 tony]$ bash /tmp/xfusioncorp.sh
Welcome To KodeKloud
```

And just like that, we've verified that others can read the file and execute it as well. But let's go back to what happens if we can't _read_ the file - trying to run a shell script without read permissions produces the following error:

```console
[tony@stapp01 ~]$ sudo chmod a-r /tmp/xfusioncorp.sh
[tony@stapp01 ~]$ bash /tmp/xfusioncorp.sh
bash: /tmp/xfusioncorp.sh: Permission denied
```

So why is that? From what I've gathered by cruising a couple of StackOverflow answers, your shell needs to *interpret* the script and thus has to read it into memory under your user's session in order to execute it, thus the error if the read bit is removed. However, you *CAN* do `execute` and not `read` on compiled binaries, i.e. building a program in C/C++ or another compiled language and running it like the following:

```console
[tony@stapp01 ~]$ cat /tmp/hello.c
#include<stdio.h>

int main() {
        printf("Hello, blog readers!\n");
        return 0;
}

[tony@stapp01 ~]$ gcc -o hello /tmp/hello.c
[tony@stapp01 ~]$ ls -la hello
-rwxr-xr-x 1 tony tony 17512 Jun  6 02:43 hello
[tony@stapp01 ~]$ chmod a-r hello
[tony@stapp01 ~]$ ls -la hello
--wx--x--x 1 tony tony 17512 Jun  6 02:43 hello
[tony@stapp01 ~]$ ./hello
Hello, blog readers!
```

## Day 5: SELinux Install && Configuration
> [!QUOTE]+ Problem Prompt
> Following a security audit, the xFusionCorp Industries security team has opted to enhance application and server security with SELinux. To initiate testing, the following requirements have been established for App server 2 in the Stratos Datacenter:
>
> 1. Install the required SELinux packages.  
> 2. Permanently disable SELinux for the time being; it will be re-enabled after necessary configuration changes.  
> 3. No need to reboot the server, as a scheduled maintenance reboot is already planned for tonight.  
> 4. Disregard the current status of SELinux via the command line; the final status after the reboot should be disabled.  
{icon="circle-question"}

```console
ssh steve@stapp02
dnf search selinux
```
```console
selinux-policy.noarch : SELinux policy configuration
selinux-policy-automotive.noarch : SELinux automotive policy
selinux-policy-devel.noarch : SELinux policy development files
selinux-policy-doc.noarch : SELinux policy documentation
selinux-policy-minimum.noarch : SELinux minimum policy
selinux-policy-mls.noarch : SELinux MLS policy
selinux-policy-sandbox.noarch : SELinux sandbox policy
selinux-policy-targeted.noarch : SELinux targeted policy
```

man selinux: https://man7.org/linux/man-pages/man8/selinux.8.html
> The /etc/selinux/config configuration file controls whether
> SELinux is enabled or disabled, and if enabled, whether SELinux
> operates in permissive mode or enforcing mode.  The SELINUX
> variable may be set to any one of disabled, permissive, or
> enforcing to select one of these options.

sudo vi /etc/selinux/config, change SELINUX to disabled
verify: egrep -i '^selinux=' /etc/selinux/config

## Day 6: Create Cron Job
> [!QUOTE]+ Problem Prompt
> The Nautilus system admins team has prepared scripts to automate several day-to-day tasks. They want them to be deployed on all app servers in Stratos DC on a set schedule. Before that they need to test similar functionality with a sample cron job. Therefore, perform the steps below:
> 
> a. Install cronie package on all Nautilus app servers and start crond service.  
> b. Add a cron */5 * * * * echo hello > /tmp/cron_text for root user.
{icon="circle-question"}

```console
[tony@stapp01 ~]$ dnf list --installed | grep cronie
cronie.x86_64                       1.5.7-14.el9                  @baseos       
cronie-anacron.x86_64               1.5.7-14.el9                  @baseos
```

Installed - so verify service status
Running - so add the crontab
sudo crontab -e
verify: sudo crontab -l
alternatively sudo ls /var/spool/cron/ to see the individual crontabs


## Day 7: Linux SSH Authentication
> [!QUOTE]+ Problem Prompt
> The system admins team of xFusionCorp Industries has set up some scripts on jump host that run on regular intervals and perform operations on all app servers in Stratos Datacenter. To make these scripts work properly we need to make sure the thor user on jump host has password-less SSH access to all app servers through their respective sudo users (i.e tony for app server 1). Based on the requirements, perform the following:
>  
> Set up a password-less authentication from user thor on jump host to all app servers through their respective sudo users.
{icon="circle-question"}

ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa
(no passphrase)

ssh-copy-id -i ~/.ssh/id_rsa tony@stapp01
verify the key worked: ssh tony@stapp01
if no pw prompt, you're good
repeat this for all hosts

bonus: disabling password auth entirely
sudo grep -i password /etc/ssh/sshd_config
sudo vi /etc/ssh/sshd_config
Modify PasswordAuthentication to no
sudo systemctl restart sshd
do this for all hosts


## Day 8: Install Ansible
> [!QUOTE]+ Problem Prompt
> During the weekly meeting, the Nautilus DevOps team discussed about the automation and configuration management solutions that they want to implement. While considering several options, the team has decided to go with Ansible for now due to its simple setup and minimal pre-requisites. The team wanted to start testing using Ansible, so they have decided to use jump host as an Ansible controller to test different kind of tasks on rest of the servers.
> 
> Install ansible version 4.8.0 on Jump host using pip3 only. Make sure Ansible binary is available globally on this system, i.e all users on this system are able to run Ansible commands.
{icon="circle-question"}

sudo pip3 install ansible==4.8.0
verify:
which ansible
ansible --version

Done!

## Day 9: MariaDB Troubleshooting
> [!QUOTE]+ Problem Prompt
> There is a critical issue going on with the Nautilus application in Stratos DC. The production support team identified that the application is unable to connect to the database. After digging into the issue, the team found that mariadb service is down on the database server.
> 
> Look into the issue and fix the same.
{icon="circle-question"}

ssh peter@stdb01

check status, try to restart as an easy fix
check journalctl
check logs: sudo cat /var/log/mariadb/mariadb.log

sudo ls -la /run/mariadb/
```
[peter@stdb01 ~]$ sudo ls -la /run/mariadb/
total 0
drwxr-xr-x  2 root mysql  40 Dec 31 02:25 .
drwxr-xr-x 17 root root  420 Dec 31 02:25 ..
```
sudo chown -R mysql:mysql /run/mariadb/
sudo systemctl restart mariadb
systemctl status mariadb to verify the fix worked!

## Day 10: Linux Bash Scripts
> [!QUOTE]+ Problem Prompt
> The production support team of xFusionCorp Industries is working on developing some bash scripts to automate different day to day tasks. One is to create a bash script for taking websites backup. They have a static website running on App Server 1 in Stratos Datacenter, and they need to create a bash script named media_backup.sh which should accomplish the following tasks. (Also remember to place the script under /scripts directory on App Server 1).
> 
> - Create a zip archive named xfusioncorp_media.zip of /var/www/html/media directory.  
> - Save the archive in /backup/ on App Server 1. This is a temporary storage, as backups from this location will be clean on weekly basis. Therefore, we also need to save this backup archive on Nautilus Backup Server.  
> - Copy the created archive to Nautilus Backup Server server in /backup/ location.  
> - Please make sure script won't ask for password while copying the archive file. Additionally, the respective server user (for example, tony in case of App Server 1) must be able to run it.  
> - Do not use sudo inside the script.  
> 
> #### **Note:**  
> The zip package must be installed on given App Server before executing the script. This package is essential for creating the zip archive of the website files. Install it manually outside the script.
{icon="circle-question"}

Pseudocode for the script:
1. Recursively zip a folder, saving in /backup locally
2. Copy the zip over to the backup server, saving in /backup remotely
3. Ensure that it is executable by tony and doesn't ask for a password

Proposed solution: set up passwordless SSH with the script having a setuid bit set

first, check if zip is around:
```
[tony@stapp01 ~]$ dnf list --installed | grep zip
bzip2-libs.x86_64                  1.0.8-10.el9                @System          
gzip.x86_64                        1.12-1.el9                  @System
```

sudo dnf install zip vim
let's check help page to get the command format for use inside the script: zip -h2
test it to make sure it works outside of script: zip -r /backup/xfusioncorp_media /var/www/html/media

good, now to setup passwordless SSH
gen a key:
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa
(no passphrase)

now copy id and verify it works
ssh-copy-id -i ~/.ssh/id_rsa clint@stbkp01
ssh clint@stbkp01

for the final piece - let's copy a file to /backup on remote
```
scp --help
scp -i ~/.ssh/id_rsa test.txt clint@stbkp01:/backup/test.txt
```

start writing our script - sudo vim /scripts/media_backup.sh
```
#!/usr/bin/env bash

echo 'hello world'
```


So that's not going to work - there's a massive reason that I discovered as a rabbit hole while trying to follow that solution. https://unix.stackexchange.com/questions/364/allow-setuid-on-shell-scripts/2910#2910

(This makes sense - setUID is dangerous business.)

Re-reading the prompt - I can just sudo script.sh and edit the sudoers file, instead of setting all that up. It's late.

let's set up the permissions now
```
sudo chown -R tony:tony /scripts/media_backup.sh
```

adding script content:
```bash
#!/usr/bin/env bash

echo '[+] Beginning backup...'

zip -r /backup/xfusioncorp_media /var/www/html/media
scp -i /home/tony/.ssh/id_rsa /backup/xfusioncorp_media.zip clint@stbkp01:/backup/xfusioncorp_media.zip
```

Adding tony to the sudoers: `sudo visudo`
```
tony ALL=(ALL) NOPASSWD:/scripts/media_backup.sh
```

Finally, run it to check and make sure to verify the results.
```
sudo bash /scripts/media_backup.sh
ls /backup
ssh clint@stbkp01 'ls /backup'
```