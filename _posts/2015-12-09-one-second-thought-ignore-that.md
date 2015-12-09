---
layout: post
category: blog
date: "2015-12-09 13:52 -0500"
author: JLab Skunkworks
tags: 
  - code snippet
  - shell
  - git
published: true
title: "One second thought, ignore that"
---


I have often struggled with getting git to ignore previously tracked stuff, particularly directories.  Here is how to do it, in this case removing and ignoring previously tracked .c9 config directory (assuming you have already added `.c9` to `.gitignore`.

```
git rm -r --cached .c9
git add .
git commit -am 'Goodbye, persistant directory'
git push
```