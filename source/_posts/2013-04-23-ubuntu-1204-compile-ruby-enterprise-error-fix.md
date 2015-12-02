---
layout: post
title: "Ubuntu 12.04 compile ruby enterprise error fix"
description: "a error fix for compile ree"
category: ruby
tags: ['ruby']
---

Recently I've been learn to use [Vagrant](http://vagrantup.com). to set up a production like env under ubuntu 12.04. However, when I tried to compile REE- ruby enterprise version, I caught into the following error:

``` bash
./libtool: line 983: warning: setlocale: LC_CTYPE: cannot change locale (UTF-8)
src/tcmalloc.cc:1672:54: error: conflicting declaration 'void * (* __memalign_hook)(size_t, size_t, const void *)'
/usr/include/malloc.h:183:39: error: "__memalign_hook" has a previous declaration as 'void * (* volatile __memalign_hook) (size_t, size_t, const void *)'
src/tcmalloc.cc: In function 'void PrintStats(int)':
src/tcmalloc.cc:523:47: warning: ignoring return value of 'ssize_t write(int, const void *, size_t)', declared with attribute warn_unused_result (-Wunused-result)
src/tcmalloc.cc: In function 'void {anonymous}::ReportLargeAlloc(Length, void *)';
src/tcmalloc.cc:1010:47: warning: ignoring return value of 'ssize_t write(int, const void *, size_t)', declared with attribute warn_unused_result [-Wunused-result]
make: *** (libtcmalloc_minimal_la-tcmalloc.lo) Error 1
```

I dig a little throught google and it turns out to be a very easy solution. Just redefine the fucntion name

``` bash
void *(* __memalign_hook) (size_t, size_t, const void *)
```

to be

``` bash
void *(*  volatile  __memalign_hook) (size_t, size_t, const void *)
```

in the tcmalloc.cc src file . To declare it to be volatile type function can solve that issue and keep the compiling on. Actually, I could have found the reason because as to what the log says, the issue is pretty clear. Seems like I depend on google too much..... it's not good.
