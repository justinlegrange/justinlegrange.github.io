---
title: "KodeKloud's 100 Days of DevOps: Days 1 - 10"
date: 2025-12-16 # YYYY-MM-DD
lastmod: 2026-07-01
summary: "A walkthrough for days 1 through 10 of KodeKloud's 100 Days of DevOps challenges."
draft: false
series: ["KodeKloud's 100 Days of DevOps"]
series_order: 1
categories: ["DevOps", "KodeKloud", "Sysadmin"]
tags: ["devops", "linux", "kodekloud", "bash", "sysadmin", "cron"]
---

## Introduction

Welcome to my series of blog posts centered around [KodeKloud's "100 Days of DevOps" challenge](https://kodekloud.com/100-days-of-devops)! In case you haven't looked at it before, KodeKloud's 100 Days of DevOps is a fantastic challenge series where they ask you to solve a lot of common server administration/DevOps specific tasks in interactive labs using their KodeKloud Engineer platform. 

You'll see topics like Linux administration, Git configuration, Docker changes, and Kubernetes troubleshooting through the lens of an administrator doing the work. I'm a bit late to the party (it was released a good while back!), but so far I'm loving every bit of this - it's extremely satisfying getting your hands dirty and knocking down task-over-task, learning along the way. Hands-on learning is always my preferred learning style, so something like this is right up my alley!

For all of these, I'll explain it geared towards beginners - sorry in advance to the intermediate and advanced folks - with hopefully enough background and extra information to help smooth the learning process. I do assume some familiarity with the terminal and Linux binaries, however. If you do need to learn that, there's a ton of great resources out there such as [The Linux Journey](https://labex.io/linuxjourney).  

With that said, let's get started with Day 1!

> [!IMPORTANT]+ Spoiler alert!
> In case you're squeamish about this sort of thing, there are a bunch of spoilers ahead - proceed at your own (self-learning) risk. I'll be diving into the nitty-gritty behind solutions where I can, so hopefully you'll be able to learn a thing or two.  
> 
> It's also worth noting that if you're working alongside me, you'll see different users, IP addresses, passwords, or even completely different solutions occasionally - they rotate these with each challenge spawn on most challenges.
{icon="circle-info"}

## Day 1: User with Non-Interactive Shell
> [!QUOTE]+ Problem Prompt
> To accommodate the backup agent tool's specifications, the system admin team at xFusionCorp Industries requires the creation of a user with a non-interactive shell. Here's your task:  
> 
> Create a user named mark with a non-interactive shell on App Server 3.
{icon="circle-question"}

First things first - on Linux, what is a non-interactive shell, and how do we add a user to it? There are 4 types of shells in the Linux world that I could find:
- **Interactive login** - This is the type of shell you get when you SSH into a computer, or pop open the local TTY before your window manager/display environment loads
- **Interactive, non-login** - An example of this shell type is opening a new window in something like Konsole or Terminal
- **Non-interactive, login** - This one was new to me, but the basic gist I got is that these spawn from piping input into a login shell somehow, i.e. running a command like `echo hello | ssh <server IP>`. I learned it researching for this question and stumbling upon [this StackOverflow answer](https://askubuntu.com/questions/879364/differentiate-interactive-login-and-non-interactive-non-login-shell)
- **Non-interactive, non-login** - This shell type would be used for scripts dropping into subshell

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

And the user is created! You can verify with a ton of different methods, but there's always the quick check in `/etc/passwd`. The last field is the one that shows the login shell - for `mark`, that's pointed correctly to `/sbin/nologin`:
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

You can edit the line in whatever text editor/stream editor you fancy, in this case I'm running the change through `sed` with the `-i` flag to edit the file in place, and then restarting the service after:

```console
[tony@stapp01 ~]$ sudo sed -i "s/PermitRootLogin yes/PermitRootLogin no/g" /etc/ssh/sshd_config
[tony@stapp01 ~]$ sudo systemctl restart sshd
```

To verify our changes were successful, we can try to SSH with the `root` account and be denied even if you provide the correct password. Repeat the process for the other servers and you should be set!


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

## Day 5: SELinux Install and Configuration
> [!QUOTE]+ Problem Prompt
> Following a security audit, the xFusionCorp Industries security team has opted to enhance application and server security with SELinux. To initiate testing, the following requirements have been established for App server 2 in the Stratos Datacenter:
>
> 1. Install the required SELinux packages.  
> 2. Permanently disable SELinux for the time being; it will be re-enabled after necessary configuration changes.  
> 3. No need to reboot the server, as a scheduled maintenance reboot is already planned for tonight.  
> 4. Disregard the current status of SELinux via the command line; the final status after the reboot should be disabled.  
{icon="circle-question"}

This is another straightforward one - we need to check if SELinux is present on the system, and if not we install it - and then make sure it's permanently in the disabled state. 

Let's start by finding out what we have to work with. SSH over to the server and check on the installation status of SELinux with DNF, like so:

```console
[steve@stapp02 ~]$ dnf list --installed | grep -i selinux
libselinux.x86_64                              3.6-1.el9                        @System       
libselinux-utils.x86_64                        3.6-1.el9                        @baseos
```

We have a few libraries already on the system, but it doesn't match the full install package name:
```console
[steve@stapp02 ~]$ dnf search selinux
[...]
selinux-policy.noarch : SELinux policy configuration
selinux-policy-automotive.noarch : SELinux automotive policy
selinux-policy-devel.noarch : SELinux policy development files
selinux-policy-doc.noarch : SELinux policy documentation
selinux-policy-minimum.noarch : SELinux minimum policy
selinux-policy-mls.noarch : SELinux MLS policy
selinux-policy-sandbox.noarch : SELinux sandbox policy
selinux-policy-targeted.noarch : SELinux targeted policy
[...]
```

So we need to grab the `selinux-policy-*` packages that match our needs. We'll default to `selinux-policy`, but it's worth investigating which one suits your use case for installs _outside_ of this lab. Install the packages with `$ sudo dnf install -y selinux-policy` and then we can work on configuring the SELinux execution policy to be set to `Disabled`.

In order to do that, we need to change `/etc/selinux/config` - [here's what the manpage](https://man7.org/linux/man-pages/man8/selinux.8.html) has to say about that:
> [!QUOTE]+ SELinux Manpage
> The /etc/selinux/config configuration file controls whether
> SELinux is enabled or disabled, and if enabled, whether SELinux
> operates in permissive mode or enforcing mode.  The SELINUX
> variable may be set to any one of disabled, permissive, or
> enforcing to select one of these options.

So now we just need to edit the file using whichever editor you're comfortable with. It's easy to do with `sed`, so that's what I'll go with. I always start by checking the command by streaming it, and then once I'm confident it's doing the intended edit I commit the change by running it in-place on the file with `sed -i`:
```console
[steve@stapp02 ~]$ sed 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:

[...]

#
#    grubby --update-kernel ALL --remove-args selinux
#
SELINUX=disabled
# SELINUXTYPE= can take one of these three values:
#     targeted - Targeted processes are protected,
#     mls - Multi Level Security protection.
SELINUXTYPE=mls
[steve@stapp02 ~]$ sudo sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

Do note that by doing it with `sed`, you end up changing the comments as well - anything that matches `SELINUX=enforcing` gets swapped over, so some of the documentation in-file might not make sense afterwards. Now all that's left is verifying that our change went through using `egrep`:

```console
[steve@stapp02 ~]$ egrep -i '^selinux=' /etc/selinux/config
SELINUX=disabled
```

## Day 6: Create Cron Job
> [!QUOTE]+ Problem Prompt
> The Nautilus system admins team has prepared scripts to automate several day-to-day tasks. They want them to be deployed on all app servers in Stratos DC on a set schedule. Before that they need to test similar functionality with a sample cron job. Therefore, perform the steps below:
> 
> a. Install cronie package on all Nautilus app servers and start crond service.  
> b. Add a cron */5 * * * * echo hello > /tmp/cron_text for root user.
{icon="circle-question"}

So first and foremost, what is a cron job anyways? As you probably guessed from the problem prompt context, cron jobs are time-based automated tasks that the OS will run without any interaction. They run on a schedule that we define, running any tasks that we want covered - whether that's quick single-line commands or running an entire script that we've written. Cronjobs are best suited for tasks that are repeatable - if you want to just schedule a single run task, the `at` binary covers that use case a bit better - and is set up either by modifying the system-wide configuration files or through a user's personal crontab.

First, SSH in to whichever server you'd like to start with - I'll be using `tony@stapp01` in the examples below, but we'll have to do this to all three app servers in order to complete the task. Once we're signed in, check what we have installed using `dnf`:

```console
[tony@stapp01 ~]$ dnf list --installed | egrep -i "cronie*"
[tony@stapp01 ~]$
```

As the problem prompt mentions, we now need to install the `cronie` package and start the service in order to have `crond` listening for tasks:
```console
[tony@stapp01 ~]$ sudo dnf install -y cronie
Last metadata expiration check: 0:00:56 ago on Tue Jun 30 21:10:38 2026.
Dependencies resolved.
=================================================================================================================
 Package                     Architecture        Version                               Repository           Size
=================================================================================================================
Installing:
 cronie                      x86_64              1.5.7-16.el9                          baseos              119 k
Installing dependencies:
 cronie-anacron              x86_64              1.5.7-16.el9                          baseos               31 k
 crontabs                    noarch              1.11-26.20190603git.el9               baseos               19 k

 [...]

 Installed:
  cronie-1.5.7-16.el9.x86_64    cronie-anacron-1.5.7-16.el9.x86_64    crontabs-1.11-26.20190603git.el9.noarch   

Complete!
[tony@stapp01 ~]$ sudo systemctl start crond.service
[tony@stapp01 ~]$ sudo systemctl status crond.service
● crond.service - Command Scheduler
     Loaded: loaded (/usr/lib/systemd/system/crond.service; enabled; preset: enabled)
     Active: active (running) since Wed 2026-07-01 16:17:54 UTC; 5s ago
   Main PID: 29366 (crond)
      Tasks: 1 (limit: 404712)
     Memory: 964.0K
        CPU: 2ms
     CGroup: /system.slice/crond.service
             └─29366 /usr/sbin/crond -n
```

Perfect - now that we have the `crond` service running, we can start to add in `crontabs` for the user that define the jobs. In this case, we need to define the job for the `root` user so we run `sudo crontab -e`, press `I` to go into insert mode, and paste in the job from the prompt. We'll then exit the file by hitting `Escape`, followed by `:wq`. Verify the change with `sudo crontab -l` to see the changes:

```console
[tony@stapp01 ~]$ sudo crontab -e
no crontab for root - using an empty one
crontab: installing new crontab

[tony@stapp01 ~]$ sudo crontab -l
*/5 * * * * echo hello > /tmp/cron_text
```

And we're finished with the task - now we just need to repeat the process for each of the other app servers!

But being done with the task doesn't mean that's all the info I can give - let's look at an alternative way to check individual crontabs. By looking into `/var/spool/cron/`, we can see the individual crontabs for each user. Right now, we only have the one for `root`:

```console
[tony@stapp01 ~]$ sudo ls -la /var/spool/cron
total 16
drwx------ 2 root root 4096 Jun 30 21:13 .
drwxr-xr-x 1 root root 4096 Jun 30 21:11 ..
-rw------- 1 root root   40 Jun 30 21:13 root

[tony@stapp01 ~]$ sudo cat /var/spool/cron/root
*/5 * * * * echo hello > /tmp/cron_text
```

Here's what it ends up looking like if we make a crontab for `tony`, using the same job as before but with a modified file name:
```console
[tony@stapp01 ~]$ crontab -e
no crontab for tony - using an empty one
crontab: installing new crontab

[tony@stapp01 ~]$ sudo ls -la /var/spool/cron
total 20
drwx------ 2 root root 4096 Jun 30 21:15 .
drwxr-xr-x 1 root root 4096 Jun 30 21:11 ..
-rw------- 1 root root   40 Jun 30 21:13 root
-rw------- 1 tony tony   45 Jun 30 21:15 tony
```

The sharp-eyed of you may have noticed that the permissions are set to read and write, only by their respective users, with the `/var/spool/cron` directory being owned by `root`. This is important for both privacy and security reasons - if a user could randomly insert a scheduled job into another user's `crontab`, that would be code-execution-as-a-service under their user permissions. That would be bad, but it would be infinitely worse if that user was `root` - it would be total compromise of the whole box!

## Day 7: Linux SSH Authentication
> [!QUOTE]+ Problem Prompt
> The system admins team of xFusionCorp Industries has set up some scripts on jump host that run on regular intervals and perform operations on all app servers in Stratos Datacenter. To make these scripts work properly we need to make sure the thor user on jump host has password-less SSH access to all app servers through their respective sudo users (i.e tony for app server 1). Based on the requirements, perform the following:
>  
> Set up a password-less authentication from user thor on jump host to all app servers through their respective sudo users.
{icon="circle-question"}

As in the last section, I'll be showing the setup for `tony` on `App Server 1` - feel free to start with whichever server you like. First we need to generate a new SSH key with no passphrase (_always_ add a passphrase on production systems for better security):

```console
[thor@jump-host ~]$ ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519
Generating public/private ed25519 key pair.
Created directory '/home/thor/.ssh'.
Enter passphrase for "/home/thor/.ssh/id_ed25519" (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/thor/.ssh/id_ed25519
Your public key has been saved in /home/thor/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:ZICM+0klSXARcVzPgwCOmmGcgBD9OBgD1yTOe1SUMgc thor@jump-host
The key's randomart image is:
+--[ED25519 256]--+
|Oo+BE@*o.        |
|*++=Oo=o +       |
|.B++o*  + +      |
|o++oo  o   .     |
|o .+..  S        |
|   .o            |
|                 |
|                 |
|                 |
+----[SHA256]-----+
```

Now that we've generated that file, we need to add the key to the server using `tony`'s account and password with `ssh-copy-id`, passing in our key with the `-i` flag:

```console
[thor@jump-host ~]$ ssh-copy-id -i ~/.ssh/id_ed25519 tony@stapp01
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/thor/.ssh/id_ed25519.pub"
The authenticity of host 'stapp01 (10.244.81.43)' can't be established.
ED25519 key fingerprint is SHA256:TBCxYGCebV3DdIFnCgvNchdAx9G/r5ol8gO5ZYR+I7M.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
tony@stapp01's password: 

Number of key(s) added: 1

Now try logging into the machine, with: "ssh -i /home/thor/.ssh/id_ed25519 'tony@stapp01'"
and check to make sure that only the key(s) you wanted were added.
```

Finally, we only need to verify that copying the key to `tony`'s account worked. If we try SSH'ing over using the key, it shouldn't prompt us about the account's password:
```console
thor@jump-host ~$ ssh -i ~/.ssh/id_ed25519 tony@stapp01
[tony@stapp01 ~]$
```

As with the other section, we just have to repeat this for all of the app servers to complete the task as described. But to truly confirm that passwordless SSH is functioning, let's take it a step further and _completely disable_ password authentication on the remote host. To start, SSH over to `stapp01` as `tony` and let's examine the password policies for SSH. We can find the SSH password parameters in `/etc/ssh/sshd_config` by looking for "password":

```console
[tony@stapp01 ~]$ sudo grep -i password /etc/ssh/sshd_config
[...]
# To disable tunneled clear text passwords, change to no here!
#PasswordAuthentication yes
#PermitEmptyPasswords no
[...]
```

I've trimmed some output off that, but you get the gist - lines with `#` at the start are comments, so currently `PasswordAuthentication` is unset, defaulting to this [according to the man page](https://man7.org/linux/man-pages/man5/sshd_config.5.html). `Ctrl+F` for "PasswordAuthentication" or you can check your system by running `man 5 sshd_config` in a terminal to find this section:
```
PasswordAuthentication
        Specifies whether password authentication is allowed.  The
        default is yes.
```

Now we just need to uncomment it and set it to "no" in order to disable the SSH daemon from allowing password authentication for all accounts. You can do this in any editor - as usual, I default to `sed` where possible to make it easy to see my changes for the blog. First, check our `sed` syntax to make sure it'll change only the wanted lines:

```console
[tony@stapp01 ~]$ sudo sed 's/#PermitEmptyPasswords no/PermitEmptyPasswords yes/g' /etc/ssh/sshd_config|grep PermitEmpty
PermitEmptyPasswords yes
```

Once you're happy with the output, just apply the change and restart the SSH service to enforce:

```console
[tony@stapp01 ~]$ sudo sed -i 's/#PermitEmptyPasswords no/PermitEmptyPasswords yes/g' /etc/ssh/sshd_config
[tony@stapp01 ~]$ sudo systemctl restart sshd
```

## Day 8: Install Ansible
> [!QUOTE]+ Problem Prompt
> During the weekly meeting, the Nautilus DevOps team discussed about the automation and configuration management solutions that they want to implement. While considering several options, the team has decided to go with Ansible for now due to its simple setup and minimal pre-requisites. The team wanted to start testing using Ansible, so they have decided to use jump host as an Ansible controller to test different kind of tasks on rest of the servers.
> 
> Install ansible version 4.8.0 on Jump host using pip3 only. Make sure Ansible binary is available globally on this system, i.e all users on this system are able to run Ansible commands.
{icon="circle-question"}

This task is extremely straightforward - in order to install a binary globally with pip, we just need to run the `pip install` with `sudo` permissions. We can specify a version using the syntax shown:

```console
[thor@jump-host ~]$ sudo pip3 install ansible==4.8.0
Collecting ansible==4.8.0
  Downloading ansible-4.8.0.tar.gz (36.1 MB)
     |████████████████████████████████| 36.1 MB 9.1 MB/s
[...]
Successfully installed MarkupSafe-3.0.3 PyYAML-6.0.3 ansible-4.8.0 ansible-core-2.11.12 cffi-2.0.0 cryptography-49.0.0 jinja2-3.1.6 packaging-26.2 pycparser-2.23 resolvelib-0.5.4
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
```

Now all we need to do is verify that we have the binary accessible under `thor`'s account - since we didn't install `ansible` under this account (i.e. used `sudo` to install), if it's actually a global install it'll work:

```console
[thor@jump-host ~]$ which ansible
/usr/local/bin/ansible
[thor@jump-host ~]$ ansible --version
ansible [core 2.11.12] 
  config file = None
  configured module search path = ['/home/thor/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/lib/python3.9/site-packages/ansible
  ansible collection location = /home/thor/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/local/bin/ansible
  python version = 3.9.19 (main, Jun 11 2024, 00:00:00) [GCC 11.4.1 20231218 (Red Hat 11.4.1-3)]
  jinja version = 3.1.6
  libyaml = True
```

Done!

## Day 9: MariaDB Troubleshooting
> [!QUOTE]+ Problem Prompt
> There is a critical issue going on with the Nautilus application in Stratos DC. The production support team identified that the application is unable to connect to the database. After digging into the issue, the team found that mariadb service is down on the database server.
> 
> Look into the issue and fix the same.
{icon="circle-question"}

This one's a big jump from the `pip install` in Day 8. Let's SSH in as `peter` to `stdb01` and let's see what we have to work with and compare it against the info we got in the problem prompt. We'll check the service status initially, then move on to more complicated things as we go. First, the service status check:

```console
[peter@stdb01 ~]$ sudo systemctl status mariadb.service
○ mariadb.service - MariaDB 10.5 database server
     Loaded: loaded (/usr/lib/systemd/system/mariadb.service; disabled; preset: disabled)
     Active: inactive (dead)
       Docs: man:mariadbd(8)
             https://mariadb.com/kb/en/library/systemd/
```

As expected, the service is down, shown by the `Active: inactive (dead)` status line. Cross our fingers and try restarting the service for an (hopefully) easy fix:

```console
[peter@stdb01 ~]$ sudo systemctl restart mariadb.service
Job for mariadb.service failed because the control process exited with error code.
See "systemctl status mariadb.service" and "journalctl -xeu mariadb.service" for details.
```

No luck - let's check the journal and see what it has to say by running `journalctl -xeu mariadb.service`:

```console
[peter@stdb01 ~]$ sudo journalctl -xeu mariadb.service
[...]
Jun 30 21:50:11 stdb01 systemd[1]: mariadb.service: Main process exited, code=exited, status=1/FAILURE
░░ Subject: Unit process exited
░░ Defined-By: systemd
░░ Support: https://access.redhat.com/support
░░ 
░░ An ExecStart= process belonging to unit mariadb.service has exited.
░░ 
░░ The process' exit code is 'exited' and its exit status is 1.
Jun 30 21:50:11 stdb01 systemd[1]: mariadb.service: Failed with result 'exit-code'.
░░ Subject: Unit failed
░░ Defined-By: systemd
░░ Support: https://access.redhat.com/support
░░ 
░░ The unit mariadb.service has entered the 'failed' state with result 'exit-code'.
Jun 30 21:50:11 stdb01 systemd[1]: Failed to start MariaDB 10.5 database server.
```

So there's not a lot of data - just some output about the service trying to start, exiting, and then sending back an exit code. Since that's not super helpful, let's check out the MariaDB logs specifically. We can find them under `/var/log/mariadb`:

```console
[peter@stdb01 ~]$ sudo ls -la /var/log/mariadb/
total 16
drwxr-x--- 1 mysql mysql 4096 Jun 30 19:57 .
drwxr-xr-x 1 root  root  4096 Jun 10 09:03 ..
-rw-rw---- 1 mysql mysql 1752 Jun 30 21:50 mariadb.log
```

We've only got one file - inspecting the log, we get a lot of info, but a few lines stand out. Once again, I've trimmed off the extra output to highlight the interesting bits:

```console
[peter@stdb01 ~]$ sudo cat /var/log/mariadb/mariadb.log
2026-06-30 21:50:11 0 [Note] Starting MariaDB 10.5.29-MariaDB source revision c461188ca6ad6ec3a54201eb87ebd75797d296df server_uid 3vnZv3XgZysmu2QK2X2ffqd2MPQ= as process 30973

[...]

2026-06-30 21:50:11 0 [Note] Server socket created on IP: '::'.
2026-06-30 21:50:11 0 [ERROR] mariadbd: Can't create/write to file '/run/mariadb/mariadb.pid' (Errcode: 13 "Permission denied")
2026-06-30 21:50:11 0 [ERROR] Can't start server: can't create PID file: Permission denied
```

We're getting permissions errors on file creation - let's check the ownership of the directory in question and see what that looks like:

```
[peter@stdb01 ~]$ sudo ls -la /run/mariadb/
total 0
drwxr-xr-x  2 root mysql  40 Dec 31 02:25 .
drwxr-xr-x 17 root root  420 Dec 31 02:25 ..
```

The folder is owned by `root`, with the group set to `mysql`. There are two methods to finish this task - one being more graceful. The first, less graceful method is to just `chown` the entire directory so that the user can write the `mariadb.pid` file. On restarting the service, we don't have the errors any more: 

```console
[peter@stdb01 ~]$ sudo chown -R mysql:mysql /run/mariadb/
[peter@stdb01 ~]$ sudo systemctl restart mariadb
[peter@stdb01 ~]$
```

The other, more graceful method is to enable group writing on the folder, without changing the ownership structure. In order to do this, we need to `chmod` the folder so that the `mysql` group has write permissions and restart the service, allowing `mariadb.pid` to be created:

```console
[peter@stdb01 ~]$ sudo chmod g+w /run/mariadb/
[peter@stdb01 ~]$ sudo systemctl restart mariadb
[peter@stdb01 ~]$ sudo ls -la /run/mariadb
total 4
drwxrwxr-x  2 root  mysql  60 Jul  1 18:11 .
drwxr-xr-x 22 root  root  520 Jul  1 17:31 ..
-rw-rw----  1 mysql mysql   6 Jul  1 18:11 mariadb.pid
```

Finally, we just need to verify the fix worked by checking the service status:

```console
[peter@stdb01 ~]$ sudo systemctl status mariadb.service
● mariadb.service - MariaDB 10.5 database server
     Loaded: loaded (/usr/lib/systemd/system/mariadb.service; disabled; preset: disabled)
     Active: active (running) since Tue 2026-06-30 21:56:43 UTC; 23s ago
       Docs: man:mariadbd(8)
             https://mariadb.com/kb/en/library/systemd/
    Process: 32745 ExecStartPre=/usr/libexec/mariadb-check-socket (code=exited, status=0/SUCCESS)
    Process: 32777 ExecStartPre=/usr/libexec/mariadb-prepare-db-dir mariadb.service (code=exited, status=0/SUCCE>
    Process: 32871 ExecStartPost=/usr/libexec/mariadb-check-upgrade (code=exited, status=0/SUCCESS)
   Main PID: 32836 (mariadbd)
     Status: "Taking your SQL requests now..."
      Tasks: 28 (limit: 404712)
     Memory: 58.9M
        CPU: 171ms
     CGroup: /system.slice/mariadb.service
             └─32836 /usr/libexec/mariadbd --basedir=/usr
```

## Day 10: Linux Bash Scripts
> [!QUOTE]+ Problem Prompt
> The production support team of xFusionCorp Industries is working on developing some bash scripts to automate different day to day tasks. One is to create a bash script for archiving website content files. They have a static website running on App Server 2 in Stratos Datacenter, and they need to create a bash script named beta_archive.sh which should accomplish the following tasks. (Also remember to place the script under the /scripts directory on App Server 2).
> 
> A.) Create a zip archive named xfusioncorp_beta.zip of /var/www/html/beta directory.  
> B.) Save the archive in the /archives/ directory on the App Server 2. This is a temporary storage, as archives from this location will be cleaned on a weekly basis. Therefore, the archive should also be copied to the Nautilus Storage Server so it can be retrieved later for validation purposes.  
> C.) Copy the created archive to the Nautilus Storage Server server in the /archives/ location.  
> D.) Please make sure script won't ask for password while copying the archive file. Additionally, the respective server user (for example, tony in case of App Server 1) must be able to run it.  
> E.) Do not use sudo inside the script.  
> 
> **Note**:
> The zip package must be installed on given App Server before executing the script. This package is essential for creating the zip archive of the website files. Install it manually outside the script.
{icon="circle-question"}

That's a lot of information - let's start off by parsing out a summary of the requirements the for the script:
1. Recursively zip a folder, saving in /archives locally
2. Copy the zip over to the storage server, saving in /archives remotely
3. Ensure that it is executable by steve and doesn't ask for a password

From that, my first proposed solution was to set up passwordless SSH, with the script having the `setUID` bit set so that it would work with `sudo` permissions always. It _sounds_ like a good solution, if you're me and working on this well into the night - but there's a very big problem overall. Linux, by default, ignores the `setUID` bit on interpreted executables for security reasons - and researching that very point lead me to [this wonderful writeup](https://unix.stackexchange.com/questions/364/allow-setuid-on-shell-scripts/2910#2910) that describes it. Essentially, this means I had to go back to the drawing board, as shown by my notes:

```
Re-reading the prompt - I can just run `sudo script.sh` and edit the sudoers file if needed, instead of setting all that up. It's late.
```

Let's use that as a jumping off point. We SSH in as `steve` to `stapp02` and check if editing the `sudoers` file is needed on the host:

```console
[steve@stapp02 ~]$ sudo -l
Matching Defaults entries for steve on stapp02:
    !visiblepw, always_set_home, match_group_by_gid, always_query_group_plugin, env_reset, env_keep="COLORS
    DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS", env_keep+="MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS
    LC_CTYPE", env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES", env_keep+="LC_MONETARY
    LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE", env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET
    XAUTHORITY", secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User steve may run the following commands on stapp02:
    (ALL) ALL
    (ALL) NOPASSWD: ALL
```

That entry means that by default, `steve` can run all commands on the box with no password required under `sudo` privileges. No `sudoers` editing required, so we move on to confirming the commands that will go into the script. First, we need to check if `zip` is already installed:
```
[steve@stapp02 ~]$ dnf list --installed | grep zip
bzip2-libs.x86_64                              1.0.8-11.el9                     @System          
gzip.x86_64                                    1.12-1.el9                       @System          
unzip.x86_64                                   6.0-59.el9                       @baseos          
zip.x86_64                                     3.0-35.el9                       @baseos
```

Since it's already present, we'll check the help page to get the command format for use inside the script. We need to do a recursive zip, i.e. grabbing all of the folders and files underneath `/var/www/html/beta`, so we look for that entry:
```console
[steve@stapp02 ~]$ zip -h2

Extended Help for Zip

See the Zip Manual for more detailed help


Zip stores files in zip archives.  The default action is to add or replace
zipfile entries.

Basic command line:
  zip options archive_name file file ...

Some examples:
  Add file.txt to z.zip (create z if needed):      zip z file.txt
  Zip all files in current dir:                    zip z *
  Zip files in current dir and subdirs also:       zip -r z .
[...]
```

So now that we have the format `zip` expects, we need to test it to make sure we get the command right and it works outside of script. The command fires flawlessly, and we have our first command of the script set up.

```console
[steve@stapp02 ~]$ zip -r /archives/xfusioncorp_beta.zip /var/www/html/beta
  adding: var/www/html/beta/ (stored 0%)
  adding: var/www/html/beta/.gitkeep (stored 0%)
  adding: var/www/html/beta/index.html (stored 0%)
```

Good - now to setup passwordless SSH. We've already done that in a previous day's tasks, so all the steps are listed out in the same code block. We create a new SSH key, copy it to the remote host, and then test SSH using it without a password prompt:
```console
[steve@stapp02 ~]$ ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519
Generating public/private ed25519 key pair.
Created directory '/home/steve/.ssh'.
Enter passphrase for "/home/steve/.ssh/id_ed25519" (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/steve/.ssh/id_ed25519
Your public key has been saved in /home/steve/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:71vwF/dqP9DdIoxA4pRXQ3wX/mCFiBMUH7txRG2Qgh8 steve@stapp02
The key's randomart image is:
+--[ED25519 256]--+
|       . =B=.o=*.|
|      + o =oE=+.o|
|     o +   +++=. |
|      . .   .= o |
|        S..o. o =|
|         ..oo..++|
|          . o.o..|
|         . . ..o |
|          o. ...o|
+----[SHA256]-----+

[steve@stapp02 ~]$ ssh-copy-id -i ~/.ssh/id_ed25519 natasha@ststor01
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/steve/.ssh/id_ed25519.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
natasha@ststor01's password: 

Number of key(s) added: 1

Now try logging into the machine, with: "ssh -i /home/steve/.ssh/id_ed25519 'natasha@ststor01'"
and check to make sure that only the key(s) you wanted were added.

[steve@stapp02 ~]$ ssh -i ~/.ssh/id_ed25519 natasha@ststor01
[natasha@ststor01 ~]$ 
logout
Connection to ststor01 closed.
```

One more down; now time for the final piece - let's copy a test file to `/archives` on the remote host. The easiest way to do this is with another SSH built-in program, `scp`. Checking the usage info, we see the same `-i identity_file` option that SSH has - meaning that we can set up the scripted `scp` call to use our passwordless SSH key to copy the file. Let's test our command by copying `test.txt` over:

```console
[steve@stapp02 ~]$ touch test.txt
[steve@stapp02 ~]$ scp
usage: scp [-346ABCOpqRrsTv] [-c cipher] [-D sftp_server_path] [-F ssh_config]
           [-i identity_file] [-J destination] [-l limit] [-o ssh_option]
           [-P port] [-S program] [-X sftp_option] source ... target

[steve@stapp02 ~]$ scp -i ~/.ssh/id_ed25519 test.txt natasha@ststor01:/archives/test.txt
test.txt                                                                       100%    0     0.0KB/s   00:00
```

We finally have all the pieces we need - passwordless SSH, our zip command, and an scp command to copy the file over - so we can finally start writing our script. We can do this in `vi`, `vim`, `nano`, or whatever else suits your fancy. The easiest way is the iterative one, so I start by building out a test program like this:

```bash
#!/usr/bin/env bash

echo 'hello world'
```

Save it, and set up the permissions now so that we own the file and can execute it, and then run the script:

```console
[steve@stapp02 ~]$ sudo chown -R steve:steve /scripts/beta_archive.sh
[steve@stapp02 ~]$ sudo chmod +x /scripts/beta_archive.sh
[steve@stapp02 ~]$ sudo bash /scripts/beta_archive.sh
hello world
```

Now that we have a basic script executing, we can start adding in our commands. Since bash processes the commands top-to-bottom, the only real concern in this small script is the ordering. It's a far cry from perfect, but it's a great starting point (more on that later).

```bash
#!/usr/bin/env bash

echo '[+] Beginning backup...'

zip -r /backup/xfusioncorp_media /var/www/html/media
scp -i /home/tony/.ssh/id_rsa /backup/xfusioncorp_media.zip clint@stbkp01:/backup/xfusioncorp_media.zip
```

Finally, we just need to run it to create the backup and copy the files over for us. As always, make sure to verify the results. We can do it without an interactive SSH session by adding the command we want to execute after our SSH command:

```
[steve@stapp02 ~]$ sudo bash /scripts/beta_archive.sh
[+] Beginning backup...
updating: var/www/html/beta/ (stored 0%)
updating: var/www/html/beta/.gitkeep (stored 0%)
updating: var/www/html/beta/index.html (stored 0%)
xfusioncorp_beta.zip                                                           100%  588     1.6MB/s   00:00

[steve@stapp02 ~]$ ls -la /archives/
total 12
drwxrwxrwx 2 root root 4096 Jul  1 14:17 .
dr-xr-xr-x 1 root root 4096 Jul  1 14:17 ..
-rw-r--r-- 1 root root  588 Jul  1 14:17 xfusioncorp_beta.zip

[steve@stapp02 ~]$ ssh natasha@ststor01 'ls /archives/'
test.txt
xfusioncorp_beta.zip
```

Now that we have a complete working script, we need to talk about where there's some room for improvements. Although this script will pass the task check on KodeKloud, here's some of the fixes that could be added with refactoring and such:
- If the user _isn't_ `steve`, the script fails - changing it to be more flexible so that it can be deployed on any app server without manual script changes
- If the SSH key _isn't_ an ED25519 key named `id_ed25519`, the script fails - we could add an automated check to get a suitable key from a provided path, or ask the user _which_ one to use in the case that there's multiple keys in `~/.ssh`
- If the `/archives/` path on either host ever changes, the script breaks - we could ask the user for input on the directory to backup
- If the user `natasha` gets switched to a service account or deleted, the script breaks - we could ask the user which account to SSH into for the SCP

Keep in mind that these refactors may be outside of the intended use case, since in the proposed scenario the script may be aimed at use with something like `cron` to help automate their backup rotation. If we're trying to write a portable tool, however, we'd want to take a look at all of these to make them more flexible to avoid fail states.

## Finish!

If you've read this far, thank you - hopefully you enjoyed and/or learned something along the way. I'm (slowly) working through writing up solutions for all 100 of the 100 Days of DevOps challenge, so keep an eye out - more of these on the way, in between security posts!