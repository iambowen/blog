---
layout: post
title: don't panic when you use git reset --hard
tag: git
categories: ["git"]
---

It's not a problem when you use to `git reset --hard` to throw away commit. But often times, you need to first check whether there's still useful stuff to keeOf course, --soft is the best choice, however, what if you use `git reset --hard` without realizing that the commit you just throw is useful. And the following should be one solution to this accident.

Given i reset the commit:

``` bash
$ @git reset --hard HEAD^@

HEAD is now at 43856d7 modify test to raise conflict

```

Then i use ref the log:

``` bash
$ @git reflog -4@

43856d7 HEAD@{0}: HEAD^: updating HEAD
3378cf7 HEAD@{1}: commit: that part is easy.

```
And i see 3378cf7 is the commit i just reset, i use git show to see commit
detail

``` bash
$ @git show 3378cf7@

commit 3378cf7f1c8a197492796b3cdb2c79f0183c9efe
Author: bowen <iambowen.m@gmail.com>
Date:   Mon Oct 31 16:26:45 2011 +0800
    that part is easy.

```
And i check out the commit:

``` bash
$ @git checkout 3378cf7f1c8a197492796b3cdb2c79f0183c9efe@

Note: checking out '3378cf7f1c8a197492796b3cdb2c79f0183c9efe'.
You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b new_branch_name

HEAD is now at 3378cf7... that part is easy.
```
Then I follow the notification to create a new branch and check log:

```bash

$ git checkout -b recover
$ git log

commit 3378cf7f1c8a197492796b3cdb2c79f0183c9efe
Author: bowen <iambowen.m@gmail.com>
Date:   Mon Oct 31 16:26:45 2011 +0800

    that part is easy.
```

And i find the commit is in this branch, then i go back to master branch merge recover branch:

```bash
$git co master
$git merge recover

Updating 43856d7..3378cf7
Fast-forward
 README   |    1 -
 whatever |    1 +
 2 files changed, 1 insertions(+), 1 deletions(-)
 create mode 100644 whatever

$ git log -1

bc. commit 3378cf7f1c8a197492796b3cdb2c79f0183c9efe
Author: bowen <iambowen.m@gmail.com>
Date:   Mon Oct 31 16:26:45 2011 +0800
    that part is easy.
```

And we can see the reseted commit is coming back, that part is easy.
