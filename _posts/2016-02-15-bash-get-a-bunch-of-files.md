---
layout: post
category: blog
date: "2016-02-15 10:40 -0500"
author: eric
tags: 
  - shell
published: true
title: "Bash: get a bunch of files"
---

Here is one way to get a bunch files with a for loop

```
#!/usr/bin/bash
volumes=(1 2 3 4 5 6 7)

for i in "${volumes[@]}"
do
  wget ftp://ftp.broad.mit.edu/pub/cmap/cmap_build02.volume${i}of7.zip
  unzip *.zip
done
```

If this is going to take a while and you don't want to wait, you could use `tmux` to start in a backgroundable session, or just use nohup:

```
chmod 750 fetchall.sh
nohup ./fetchall.sh
```