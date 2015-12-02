---
layout: post
title: "Record and Play Back Terminal Session"
description: ""
category: linux
tags: ["linux"]
---



Nowadays, lots of people would use video to record a tutorial for demonstrating. It's cool for vividly showing everything to people. The only issue is that file size of video is relatively large. From this point of view, the old way of recording actions from console might be a better idea.

`script` and `scriptreplay` are console recording/playing tools. `script` would record time and actions of current session in two different files. And `scriptreplay` would play those actions again in the console。 It's quite easy to use these tools, just like the following:

```bash
[vagrant@bogon shell]$ script -t 2>timing.log -a output.session
Script started, file is output.session
[vagrant@bogon shell]$ ls -al .
total 24
drwxrwxr-x 2 vagrant vagrant 4096 Nov  5 09:37 .
drwx------ 4 vagrant vagrant 4096 Nov  4 12:15 ..
-rw-rw-r-- 1 vagrant vagrant 4400 Nov  5 09:42 output.session
-rw-rw-r-- 1 vagrant vagrant   55 Nov  4 12:16 test.txt
-rw-rw-r-- 1 vagrant vagrant  111 Nov  5 16:21 timing.log
[vagrant@bogon shell]$ cd /tmp/
[vagrant@bogon tmp]$ ls
input.txt  ks-script-iuQ695  ks-script-iuQ695.log  output.txt  stderr  try.txt  vboxguest-Module.symvers  yum.log
[vagrant@bogon tmp]$ touch something.txt
[vagrant@bogon tmp]$ exit
exit
Script done, file is output.session
```
So the `-t` would record the time and `-a` would record actions to different files. In the command below, we also redirect the error message to `timing.log` file. 
And if we want to replay it, just use the `scriptreplay` command, like this:

```bash
scriptreplay timing.log output.session
```

Then it would play what you have done before.
The thing you need to pay attention to is that such recording just happen in a session, so actions like switching user to do something would break that, to my understanding. Personally, I would use ```su user -c 'commands'``` to solve this issue.

It might not be a feature used often times, but it probably would help if you want to share or show actions you've done to others.
