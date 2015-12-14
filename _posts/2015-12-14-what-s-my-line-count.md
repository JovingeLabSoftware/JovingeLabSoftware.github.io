---
layout: post
category: blog
date: "2015-12-14 07:46 -0500"
author: JLab Skunkworks
tags: 
  - code snippet
  - R
  - shell
published: true
title: "What's my line (count)?"
---

Ever wonder how many lines of code are in your project?  You can get an idea with the tool `cloc` (`sudo apt-get install cloc`).  Unfortunately, `cloc` doesn't know about R.  So we need to get a copy of its language defintion file and add R to it:

```Shell
cloc --read-lang-def ~/cloc_lang_def.txt
```

Then add this stanza to it:

```Shell
R
    filter remove_matches ^\s*#
    extension r
    extension R
    3rd_gen_scale 2.00
```
(The 3rd_gen_scale is arbitrary). Now you can do this:

```Shell
$ cloc $(git ls-files) --read-lang-def ~/cloc_lang_def.txt | more
     160 text files.
     132 unique files.                                          
      90 files ignored.

http://cloc.sourceforge.net v 1.60  T=0.10 s (881.1 files/s, 100363.0 lines/s)
-------------------------------------------------------------------------------
Language                     files          blank        comment           code
-------------------------------------------------------------------------------
Javascript                      62            337            694           2813
HTML                             6           1677              9           1763
CSS                              9            246            122           1240
R                                9            162            189           1006
Bourne Shell                     5             24              2             82
-------------------------------------------------------------------------------
SUM:                            91           2446           1016           6904
-------------------------------------------------------------------------------
```