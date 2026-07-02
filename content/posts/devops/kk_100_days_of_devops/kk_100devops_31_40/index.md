---
title: "KodeKloud's 100 Days of DevOps: Day 31 - 40"
date: 2026-01-07 # YYYY-MM-DD
lastMod: 2026-06-24
summary: "A walkthrough for days 31 through 40 of KodeKloud's 100 Days of DevOps challenges."
draft: true
series: ["KodeKloud's 100 Days of DevOps"]
series_order: 4
categories: ["DevOps", "KodeKloud"]
tags: ["devops", "linux", "docker", "git"]
---

## Intro

Placeholder.

## Day 31: Git Stash

> [!QUOTE]+ Problem Prompt
> The Nautilus application development team was working on a git repository /usr/src/kodekloudrepos/media present on Storage server in Stratos DC. One of the developers stashed some in-progress changes in this repository, but now they want to restore some of the stashed changes. Find below more details to accomplish this task:  
> 
> Look for the stashed changes under /usr/src/kodekloudrepos/media git repository, and restore the stash with stash@{1} identifier. Further, commit and push your changes to the origin.

thor@jumphost ~$ ssh natasha@ststor01
[natasha@ststor01 ~]$ cd /usr/src/kodekloudrepos/media/
[natasha@ststor01 media]$ sudo git status
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean
[natasha@ststor01 media]$ sudo git log
commit 14456562d509b78b16b5feecaab9c38ced0c5ff4 (HEAD -> master, origin/master)
Author: Admin <admin@kodekloud.com>
Date:   Thu Jan 8 17:48:34 2026 +0000

    initial commit

[natasha@ststor01 media]$ sudo git stash -u
No local changes to save
[natasha@ststor01 media]$ sudo git stash list
stash@{0}: WIP on master: 1445656 initial commit
stash@{1}: WIP on master: 1445656 initial commit

from man:
```
pop [--index] [-q|--quiet] [<stash>]
           Remove a single stashed state from the stash list and apply it on top of the current working tree
           state, i.e., do the inverse operation of git stash push. The working directory must match the
           index.

           Applying the state can fail with conflicts; in this case, it is not removed from the stash list.
           You need to resolve the conflicts by hand and call git stash drop manually afterwards.
```

```
[natasha@ststor01 media]$ sudo git stash pop 'stash@{1}'
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   welcome.txt

Dropped stash@{1} (d499ab2e1a4a46b7a28570cb798ae861a7661b6e)
[natasha@ststor01 media]$ sudo git add .
[natasha@ststor01 media]$ sudo git commit -m "Restoring changes"
[master 192c7ed] Restoring changes
 1 file changed, 1 insertion(+)
 create mode 100644 welcome.txt
[natasha@ststor01 media]$ sudo git push
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 16 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 308 bytes | 308.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To /opt/media.git
   1445656..192c7ed  master -> master
```

## Day 32: Git Rebase

> [!QUOTE]+ Problem Prompt
> The Nautilus application development team has been working on a project repository /opt/apps.git. This repo is cloned at /usr/src/kodekloudrepos on storage server in Stratos DC. They recently shared the following requirements with DevOps team:  
> 
> One of the developers is working on feature branch and their work is still in progress, however there are some changes which have been pushed into the master branch, the developer now wants to rebase the feature branch with the master branch without loosing any data from the feature branch, also they don't want to add any merge commit by simply merging the master branch into the feature branch. Accomplish this task as per requirements mentioned.  
> 
> Also remember to push your changes once done.
{icon="circle-question"}

thor@jumphost ~$ ssh natasha@ststor01
[natasha@ststor01 ~]$ cd /usr/src/kodekloudrepos/apps
[natasha@ststor01 apps]$ sudo git status
On branch feature
nothing to commit, working tree clean
[natasha@ststor01 apps]$ sudo git log
commit 007daebade374ac8dfac729a5bf9d7015f151cda (HEAD -> feature, origin/feature)
Author: Admin <admin@kodekloud.com>
Date:   Fri Jan 9 16:32:11 2026 +0000

    Add new feature

commit 336d57fe43fef493592beceb94c59377e28991b0
Author: Admin <admin@kodekloud.com>
Date:   Fri Jan 9 16:32:11 2026 +0000

    initial commit

from man:
```
If <branch> is specified, git rebase will perform an automatic git switch <branch> before doing
       anything else. Otherwise it remains on the current branch.

       If <upstream> is not specified, the upstream configured in branch.<name>.remote and branch.<name>.merge
       options will be used (see git-config(1) for details) and the --fork-point option is assumed. If you are
       currently not on any branch or if the current branch does not have a configured upstream, the rebase
       will abort.

       All changes made by commits in the current branch but that are not in <upstream> are saved to a
       temporary area. This is the same set of commits that would be shown by git log <upstream>..HEAD; or by
       git log 'fork_point'..HEAD, if --fork-point is active (see the description on --fork-point below); or
       by git log HEAD, if the --root option is specified.

       The current branch is reset to <upstream> or <newbase> if the --onto option was supplied. This has the
       exact same effect as git reset --hard <upstream> (or <newbase>). ORIG_HEAD is set to point at the tip
       of the branch before the reset.

       [...]

       The commits that were previously saved into the temporary area are then reapplied to the current
       branch, one by one, in order. Note that any commits in HEAD which introduce the same textual changes as
       a commit in HEAD..<upstream> are omitted (i.e., a patch already accepted upstream with a different
       commit message or timestamp will be skipped).
```

[natasha@ststor01 apps]$ sudo git rebase master feature
Successfully rebased and updated refs/heads/feature.

```
[natasha@ststor01 apps]$ sudo git push
fatal: The current branch feature has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin feature

To have this happen automatically for branches without a tracking
upstream, see 'push.autoSetupRemote' in 'git help config'.
```

[natasha@ststor01 demo]$ sudo git push origin feature --force
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 16 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 295 bytes | 295.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To /opt/demo.git
 + 5eb4816...688bebf feature -> feature (forced update)

## Day 33: Resolve Git Merge Conflicts

> [!QUOTE]+ Problem Prompt
> Sarah and Max were working on writting some stories which they have pushed to the repository. Max has recently added some new changes and is trying to push them to the repository but he is facing some issues. Below you can find more details:  
>   
> SSH into storage server using user max and password Max_pass123. Under /home/max you will find the story-blog repository. Try to push the changes to the origin repo and fix the issues. The story-index.txt must have titles for all 4 stories. Additionally, there is a typo in The Lion and the Mooose line where Mooose should be Mouse.  
>  
> Click on the Gitea UI button on the top bar. You should be able to access the Gitea page. You can login to Gitea server from UI using username sarah and password Sarah_pass123 or username max and password Max_pass123.  
>  
> Note: For these kind of scenarios requiring changes to be done in a web UI, please take screenshots so that you can share it with us for review in case your task is marked incomplete. You may also consider using a screen recording software such as loom.com to record and share your work.
{icon="circle-question"}

thor@jumphost ~$ ssh max@ststor01
max $ cd story-blog
max (master)$ git push
Username for 'http://git.stratos.xfusioncorp.com': max
Password for 'http://max@git.stratos.xfusioncorp.com': 
To http://git.stratos.xfusioncorp.com/sarah/story-blog.git
 ! [rejected]        master -> master (fetch first)
error: failed to push some refs to 'http://git.stratos.xfusioncorp.com/sarah/story-blog.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.

max (master)$ git pull
remote: Enumerating objects: 4, done.
remote: Counting objects: 100% (4/4), done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (3/3), done.
From http://git.stratos.xfusioncorp.com/sarah/story-blog
   24308bd..c0ba0a0  master     -> origin/master
Auto-merging story-index.txt
CONFLICT (add/add): Merge conflict in story-index.txt
Automatic merge failed; fix conflicts and then commit the result.

max (master)$ vi story-index.txt
Change line 1 - `1. The Lion and the Mouse`
Add 4th title below `====` marker
Delete markers + bottom text
Final file:
```
1. The Lion and the Mouse
2. The Frogs and the Ox
3. The Fox and the Grapes
4. The Donkey and the Dog
```

max (master)$ git add .
max (master)$ git commit -m "Fixing merge conflicts"
[master c453427] Fixing merge conflicts
 Committer: Linux User <max@ststor01.stratos.xfusioncorp.com>
Your name and email address were configured automatically based
on your username and hostname. Please check that they are accurate.
You can suppress this message by setting them explicitly. Run the
following command and follow the instructions in your editor to edit
your configuration file:

    git config --global --edit

After doing this, you may fix the identity used for this commit with:

    git commit --amend --reset-author

max (master)$ git push
Username for 'http://git.stratos.xfusioncorp.com': max
Password for 'http://max@git.stratos.xfusioncorp.com': 
Counting objects: 7, done.
Delta compression using up to 16 threads.
Compressing objects: 100% (7/7), done.
Writing objects: 100% (7/7), 1.17 KiB | 0 bytes/s, done.
Total 7 (delta 1), reused 0 (delta 0)
remote: . Processing 1 references
remote: Processed 1 references in total
To http://git.stratos.xfusioncorp.com/sarah/story-blog.git
   c0ba0a0..c453427  master -> master

## Day 34: Git Hook

> [!QUOTE]+ Problem Prompt
> The Nautilus application development team was working on a git repository /opt/demo.git which is cloned under /usr/src/kodekloudrepos directory present on Storage server in Stratos DC. The team want to setup a hook on this repository, please find below more details:  
> Merge the feature branch into the master branch, but before pushing your changes complete below point.  
> Create a post-update hook in this git repository so that whenever any changes are pushed to the master branch, it creates a release tag with name release-2023-06-15, where 2023-06-15 is supposed to be the current date. For example if today is 20th June, 2023 then the release tag must be release-2023-06-20. Make sure you test the hook at least once and create a release tag for today's release.  
> Finally remember to push your changes.  
> Note: Perform this task using the natasha user, and ensure the repository or existing directory permissions are not altered.  
{icon="circle-question"}

Ref: https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks

What is a hook?
```console
[natasha@ststor01 apps]$ cd ./.git/hooks
[natasha@ststor01 hooks]$ ls -la
total 72
drwxr-xr-x 2 natasha natasha 4096 Jan  9 21:59 .
drwxr-xr-x 8 natasha natasha 4096 Jan  9 22:14 ..
-rwxr-xr-x 1 natasha natasha  482 Jan  9 21:59 applypatch-msg.sample
-rwxr-xr-x 1 natasha natasha  900 Jan  9 21:59 commit-msg.sample
-rwxr-xr-x 1 natasha natasha 4726 Jan  9 21:59 fsmonitor-watchman.sample
-rwxr-xr-x 1 natasha natasha  193 Jan  9 21:59 post-update.sample
-rwxr-xr-x 1 natasha natasha  428 Jan  9 21:59 pre-applypatch.sample
-rwxr-xr-x 1 natasha natasha 1653 Jan  9 21:59 pre-commit.sample
-rwxr-xr-x 1 natasha natasha  420 Jan  9 21:59 pre-merge-commit.sample
-rwxr-xr-x 1 natasha natasha 1378 Jan  9 21:59 pre-push.sample
-rwxr-xr-x 1 natasha natasha 4902 Jan  9 21:59 pre-rebase.sample
-rwxr-xr-x 1 natasha natasha  548 Jan  9 21:59 pre-receive.sample
-rwxr-xr-x 1 natasha natasha 1496 Jan  9 21:59 prepare-commit-msg.sample
-rwxr-xr-x 1 natasha natasha 2787 Jan  9 21:59 push-to-checkout.sample
-rwxr-xr-x 1 natasha natasha 2312 Jan  9 21:59 sendemail-validate.sample
-rwxr-xr-x 1 natasha natasha 3654 Jan  9 21:59 update.sample
```

Post-update hook is a server-side hook

```console
[natasha@ststor01 ~]$ cd /usr/src/kodekloudrepos/demo
[natasha@ststor01 demo]$ git status
On branch feature
nothing to commit, working tree clean
[natasha@ststor01 demo]$ git branch
* feature
  master
[natasha@ststor01 demo]$ git log
commit abb39da2456a36e74360512d02726a179e8a6e62 (HEAD -> feature, origin/feature)
Author: Admin <admin@kodekloud.com>
Date:   Tue Jun 30 19:56:23 2026 +0000

    Add feature

commit bd592d96a5febfcc87e68ccfde2c5f6408c3d058 (origin/master, master)
Author: Admin <admin@kodekloud.com>
Date:   Tue Jun 30 19:56:23 2026 +0000

    initial commit
```

first, getting the tag and conditional right:
```console
[natasha@ststor01 hooks]$ echo "release-$(date +%Y-%m-%d)"
release-2026-01-09

[natasha@ststor01 demo]$ if [[ $(git branch --show-current) -eq "master" ]]; then echo "MASTER"; fi
MASTER
```

Creating the hook:
```
[natasha@ststor01 demo]$ vi /opt/demo.git/hooks/post-update
[natasha@ststor01 demo]$ chmod +x /opt/demo.git/hooks/post-update
```

Hook:
```bash 
echo "Executing server-side post-update hook!"

if [[ $(git branch --show-current) -eq "master" ]]; then
    tag="release-$(date +%Y-%m-%d)"
    git tag -a $tag -m "$tag"
fi
exit 0
```

We need to set up our git config, otherwise we get errors:
```console
[natasha@ststor01 demo]$ git config --global user.email "a@a"
[natasha@ststor01 demo]$ git config --global user.name "n"
```

Now we need to merge in the features and push everything
```console
[natasha@ststor01 demo]$ git checkout master
Switched to branch 'master'
Your branch is up to date with 'origin/master'.
[natasha@ststor01 demo]$ git merge feature
Updating bd592d9..abb39da
Fast-forward
 feature.txt | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 feature.txt
[natasha@ststor01 demo]$ git push
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
remote: Executing server-side post-update hook!
To /opt/demo.git
   bd592d9..abb39da  master -> master
[natasha@ststor01 demo]$ cd /opt/demo.git
[natasha@ststor01 demo.git]$ git tag -l
release-2026-06-30
```


## Day 35: Install Docker Packages and Start Docker Service

> [!QUOTE]+ Problem Prompt
> The Nautilus DevOps team aims to containerize various applications following a recent meeting with the application development team. They intend to conduct testing with the following steps:
> 
> Install docker-ce and docker compose packages on App Server 3.
> 
> Initiate the docker service.
{icon="circle-question"}

super easy - follow the install instructions: https://docs.docker.com/engine/install/centos/

```console
[banner@stapp03 ~]$ dnf list --installed | egrep "docker*"

[banner@stapp03 ~]$ sudo dnf remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
No match for argument: docker
No match for argument: docker-client
No match for argument: docker-client-latest
No match for argument: docker-common
No match for argument: docker-latest
No match for argument: docker-latest-logrotate
No match for argument: docker-logrotate
No match for argument: docker-engine
No packages marked for removal.
Dependencies resolved.
Nothing to do.
Complete!

[banner@stapp03 ~]$ sudo dnf -y install dnf-plugins-core
Last metadata expiration check: 0:06:51 ago on Tue Jun 30 20:33:36 2026.
Package dnf-plugins-core-4.3.0-25.el9.noarch is already installed.
Dependencies resolved.
=================================================================================================================
 Package                               Architecture        Version                     Repository           Size
=================================================================================================================
Upgrading:
 dnf-plugins-core                      noarch              4.3.0-26.el9                baseos               36 k
 python3-dnf-plugins-core              noarch              4.3.0-26.el9                baseos              263 k
 yum-utils

 [...]

 Upgraded:
  dnf-plugins-core-4.3.0-26.el9.noarch                python3-dnf-plugins-core-4.3.0-26.el9.noarch               
  yum-utils-4.3.0-26.el9.noarch                      

Complete!



[banner@stapp03 ~]$ sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
Adding repo from: https://download.docker.com/linux/centos/docker-ce.repo


[banner@stapp03 ~]$ sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
Docker CE Stable - x86_64                                                        106 kB/s | 2.0 kB     00:00    
Docker CE Stable - x86_64                                                        1.3 MB/s |  80 kB     00:00    
Dependencies resolved.
=================================================================================================================
 Package                             Architecture     Version                   Repository                  Size
=================================================================================================================
Installing:
 containerd.io                       x86_64           2.2.5-1.el9               docker-ce-stable            35 M
 docker-buildx-plugin                x86_64           0.35.0-1.el9              docker-ce-stable            18 M
 docker-ce                           x86_64           3:29.6.1-1.el9            docker-ce-stable            24 M
 docker-ce-cli                       x86_64           1:29.6.1-1.el9            docker-ce-stable           8.5 M
 docker-compose-plugin               x86_64           5.2.0-1.el9               docker-ce-stable           8.4 M
Installing dependencies:

[...]

Installed:
  container-selinux-4:2.245.0-1.el9.noarch               containerd.io-2.2.5-1.el9.x86_64                        
  docker-buildx-plugin-0.35.0-1.el9.x86_64               docker-ce-3:29.6.1-1.el9.x86_64                         
  docker-ce-cli-1:29.6.1-1.el9.x86_64                    docker-ce-rootless-extras-29.6.1-1.el9.x86_64           
  docker-compose-plugin-5.2.0-1.el9.x86_64               iptables-legacy-1.8.10-11.1.el9.x86_64                  
  iptables-legacy-libs-1.8.10-11.1.el9.x86_64            iptables-libs-1.8.10-11.el9.x86_64                      
  jansson-2.14-1.el9.x86_64                              libmnl-1.0.4-16.el9.x86_64                              
  libnetfilter_conntrack-1.0.9-1.el9.x86_64              libnfnetlink-1.0.1-23.el9.x86_64                        
  libnftnl-1.2.6-4.el9.x86_64                            nftables-1:1.0.9-7.el9.x86_64                           
  xz-5.2.5-8.el9.x86_64                                 

Complete!


[banner@stapp03 ~]$ sudo systemctl enable --now docker
Created symlink /etc/systemd/system/multi-user.target.wants/docker.service → /usr/lib/systemd/system/docker.service.



[banner@stapp03 ~]$ sudo systemctl status docker
● docker.service - Docker Application Container Engine
     Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; preset: disabled)
     Active: active (running) since Tue 2026-06-30 20:42:41 UTC; 19s ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 38667 (dockerd)
      Tasks: 16
     Memory: 25.9M (peak: 27.2M)
        CPU: 283ms
     CGroup: /system.slice/docker.service
             └─38667 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

[...]




[banner@stapp03 ~]$ sudo docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
4f55086f7dd0: Pull complete 
Digest: sha256:96498ffd522e70807ab6384a5c0485a79b9c7c08ca79ba08623edcad1054e62d
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
 ```

## Day 36: Deploy Nginx Container on Application Server
> [!QUOTE]+ Problem Prompt
> The Nautilus DevOps team is conducting application deployment tests on selected application servers. They require a nginx container deployment on Application Server 1. Complete the task with the following instructions:
> 
> On Application Server 1 create a container named nginx_1 using the nginx image with the alpine tag. Ensure container is in a running state.
{icon="circle-question"}

```console
[tony@stapp01 ~]$ docker run -d --name nginx_1 nginx:alpine
Unable to find image 'nginx:alpine' locally
alpine: Pulling from library/nginx
e6f31ffc071e: Pull complete 
c16defe09b2f: Pull complete 
5b429a43b8df: Pull complete 
967885d218c5: Pull complete 
ab1fd9049751: Pull complete 
ce42635eeddd: Pull complete 
01bf363d61e6: Pull complete 
c75b9c33e8b0: Pull complete 
Digest: sha256:54f2a904c251d5a34adf545a72d32515a15e08418dae0266e23be2e18c66fefa
Status: Downloaded newer image for nginx:alpine
dd06593fb59e14f0c574a424044947b0ef27f38c1822c87f114c5372938ed564
[tony@stapp01 ~]$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS     NAMES
dd06593fb59e   nginx:alpine   "/docker-entrypoint.…"   3 seconds ago   Up 2 seconds   80/tcp    nginx_1
```

## Day 37: Copy File to Docker Container

> [!QUOTE]+ Problem Prompt

{icon="circle-question"}

Placeholder.

## Day 38: Pull Docker Image

Placeholder.

## Day 39: Create a Docker Image From Container

Placeholder.

## Day 40: Docker EXEC Operations

Placeholder.
