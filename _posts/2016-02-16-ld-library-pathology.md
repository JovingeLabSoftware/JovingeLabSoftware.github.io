---
layout: post
category: blog
date: "2016-02-16 14:00 -0500"
author: eric
tags: 
  - R
  - shell
published: true
title: LD_LIBRARY_PATHology
---

Whilst trying to install `BatchJobs` for R on our HPC cluster, I encountered an error during installation of one of its dependencies, the `stringi` library:

```
Error in dyn.load(file, DLLpath = DLLpath, ...) :
  unable to load shared object '/lib64/R/library/stringi/libs/stringi.so':
  /lib64/libstdc++.so.6: version `CXXABI_1.3.8' not found (required by /lib64/R/library/stringi/libs/stringi.so)
Error: loading failed
Execution halted
```
It was pretty obvious I had the wrong version of `libstdc++.so.6`, but why?  After googling around a bit I learned that the `gcc` installation likely had its own shared libraries installed separately (not in `/lib64`), and that I needed to point my environment to those libraries.  (I'm not totally clear why the libraries in /lib64 on my system are out of sync with our version of `gcc`--5.1--but that is for another day).

So I tracked down the location of `gcc` with `which gcc` which gave me:

```
/cm/local/apps/gcc/5.1.0/bin/gcc
```
Investigating that location revealed that, sure enough, there was:

```
/cm/local/apps/gcc/5.1.0/lib64
```

Which contained `libstdc++.s0.6`.  Interrogating this libary file with `strings` revealed:

```
[eric.kort@login01 ~]$ strings /cm/local/apps/gcc/current/lib64/libstdc++.so.6 | grep CXXABI_1.3
CXXABI_1.3
CXXABI_1.3.1
CXXABI_1.3.2
CXXABI_1.3.3
CXXABI_1.3.4
CXXABI_1.3.5
CXXABI_1.3.6
CXXABI_1.3.7
CXXABI_1.3.8
CXXABI_1.3.9
```

Bingo.  So now we just need to add this location at the top of LD_LIBRARY_PATH so that the correct runtime library is located:

```
LD_LIBRARY_PATH=/cm/local/apps/gcc/current/lib64:$LD_LIBRARY_PATH
export LD_LIBRARY_PATH
```

After making that change, `stringi` installed without difficulty.

