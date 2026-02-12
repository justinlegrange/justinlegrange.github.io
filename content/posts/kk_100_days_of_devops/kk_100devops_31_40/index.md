---
title: "KodeKloud's 100 Days of DevOps: Day 31 - 40"
date: 2026-01-07 # YYYY-MM-DD
description: "A walkthrough for days 31 through 40 of KodeKloud's 100 Days of DevOps challenges."
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

## Intro

## Day 31: Git Stash

Prompt:
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

Prompt:
> The Nautilus application development team has been working on a project repository /opt/apps.git. This repo is cloned at /usr/src/kodekloudrepos on storage server in Stratos DC. They recently shared the following requirements with DevOps team:  
> 
> One of the developers is working on feature branch and their work is still in progress, however there are some changes which have been pushed into the master branch, the developer now wants to rebase the feature branch with the master branch without loosing any data from the feature branch, also they don't want to add any merge commit by simply merging the master branch into the feature branch. Accomplish this task as per requirements mentioned.  
> 
> Also remember to push your changes once done.

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

Prompt:
> Sarah and Max were working on writting some stories which they have pushed to the repository. Max has recently added some new changes and is trying to push them to the repository but he is facing some issues. Below you can find more details:  
>   
> SSH into storage server using user max and password Max_pass123. Under /home/max you will find the story-blog repository. Try to push the changes to the origin repo and fix the issues. The story-index.txt must have titles for all 4 stories. Additionally, there is a typo in The Lion and the Mooose line where Mooose should be Mouse.  
>  
> Click on the Gitea UI button on the top bar. You should be able to access the Gitea page. You can login to Gitea server from UI using username sarah and password Sarah_pass123 or username max and password Max_pass123.  
>  
> Note: For these kind of scenarios requiring changes to be done in a web UI, please take screenshots so that you can share it with us for review in case your task is marked incomplete. You may also consider using a screen recording software such as loom.com to record and share your work.

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

Prompt:
> The Nautilus application development team was working on a git repository /opt/apps.git which is cloned under /usr/src/kodekloudrepos directory present on Storage server in Stratos DC. The team want to setup a hook on this repository, please find below more details:  
>  
> Merge the feature branch into the master branch, but before pushing your changes complete below point.  
> Create a post-update hook in this git repository so that whenever any changes are pushed to the master branch, it creates a release tag with name release-2023-06-15, where 2023-06-15 is supposed to be the current date. For example if today is 20th June, 2023 then the release tag must be release-2023-06-20. Make sure you test the hook at least once and create a release tag for today's release.  
> Finally remember to push your changes.  
> Note: Perform this task using the natasha user, and ensure the repository or existing directory permissions are not altered.

[natasha@ststor01 ~]$ cd /usr/src/kodekloudrepos/apps/
[natasha@ststor01 apps]$ sudo git status
On branch feature
nothing to commit, working tree clean
[natasha@ststor01 apps]$ sudo git log
commit caa6599781bc02fc4e0b9202d5215fca60921d7c (HEAD -> feature, origin/feature)
Author: Admin <admin@kodekloud.com>
Date:   Fri Jan 9 21:59:38 2026 +0000

    Add feature

commit fa49a4e9cee276c74e9faa936b214a7ef94b1e88 (origin/master, master)
Author: Admin <admin@kodekloud.com>
Date:   Fri Jan 9 21:59:38 2026 +0000

    initial commit

[natasha@ststor01 apps]$ sudo git checkout master
Switched to branch 'master'
Your branch is up to date with 'origin/master'.
[natasha@ststor01 apps]$ sudo git merge feature
Updating fa49a4e..caa6599
Fast-forward
 feature.txt | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 feature.txt

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
[natasha@ststor01 hooks]$ cat pre-push.sample
#!/usr/bin/sh

# An example hook script to verify what is about to be pushed.  Called by "git
# push" after it has checked the remote status, but before anything has been
# pushed.  If this script exits with a non-zero status nothing will be pushed.
#
# This hook is called with the following parameters:
#
# $1 -- Name of the remote to which the push is being done
# $2 -- URL to which the push is being done
#
# If pushing without using a named remote those arguments will be equal.
#
# Information about the commits which are being pushed is supplied as lines to
# the standard input in the form:
#
#   <local ref> <local oid> <remote ref> <remote oid>
#
# This sample shows how to prevent push of commits where the log message starts
# with "WIP" (work in progress).

remote="$1"
url="$2"

zero=$(git hash-object --stdin </dev/null | tr '[0-9a-f]' '0')

while read local_ref local_oid remote_ref remote_oid
do
        if test "$local_oid" = "$zero"
        then
                # Handle delete
                :
        else
                if test "$remote_oid" = "$zero"
                then
                        # New branch, examine all commits
                        range="$local_oid"
                else
                        # Update to existing branch, examine new commits
                        range="$remote_oid..$local_oid"
                fi

                # Check for WIP commit
                commit=$(git rev-list -n 1 --grep '^WIP' "$range")
                if test -n "$commit"
                then
                        echo >&2 "Found WIP commit in $local_ref, not pushing"
                        exit 1
                fi
        fi
done

exit 0
```

first, getting the tag right:
```
[natasha@ststor01 hooks]$ echo "release-$(date +%Y-%m-%d)"
release-2026-01-09
```
sudo vi pre-push
```
tag="release-$(date +%Y-%m-%d)"
git tag -a $tag -m "$tag"
exit 0
```
[natasha@ststor01 hooks]$ cd ../../
[natasha@ststor01 apps]$ sudo git add .
[natasha@ststor01 apps]$ sudo git commit -m 'adding hook'
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

nothing to commit, working tree clean
[natasha@ststor01 apps]$ sudo git push
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To /opt/apps.git
   fa49a4e..caa6599  master -> master
[natasha@ststor01 apps]$ sudo git tag -l
release-2026-01-09

## Day 35: Install Docker Packages and Start Docker Service

## Day 36: Deploy Nginx Container on Application Server

## Day 37: Copy File to Docker Container

## Day 38: Pull Docker Image

## Day 39: Create a Docker Image From Container

## Day 40: Docker EXEC Operations