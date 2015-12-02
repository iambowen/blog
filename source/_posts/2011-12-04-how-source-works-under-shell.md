---
layout: post
title: how source works under shell
tag: linux
categories: ["linux", "shell"]
---

*Well*, it's a fare long story.During the trainning, Aimee asked me a question about
how the source command works under shell. Haven't done a deep search at that time
though it does not seem like a tough question. Now, i figure out how this stuff work, but
the trainning is over and Aimee is gone.

It's really simple for the function of source. It helps the current shell reload the variables
from script file so that they could be recongnized.In another word--read and execute
commands from the filename argument in the current shell context. Let's just cut the
crap and try some examples.

```bash
touch test.sh
chmod +x test.sh
vi test.sh  
```

*And input the following content*

```bash
A = "shit happens"
```

We just define an variable named A in the script file. ok,let's run this script with

```bash
sh test.sh && echo $A
```
which will show nothing even shit because it creates a new subshell and the invariables
created in the subshell will not show in the parent shell. Then you use

```bash
source test.sh
echo $A
```
now you can see the shit by yourself.

So? What's the difference between source and .?Well,basically, they are the same.
Instead of using source, you can use . test.sh to get the same effect of using source
even the file test.sh is not executable.It is said that use .test.sh means execute the
hidden file, definetly not working on osx.
