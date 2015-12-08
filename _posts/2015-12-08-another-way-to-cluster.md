---
layout: post
category: blog
date: "2015-12-08 14:37 -0500"
author: JLab Skunkworks
tags: null
published: true
title: Another way to cluster
---

Another simple way to parallelize.  The doMC transparently mirrors your environment on each node of the cluster.  I assume it uses black magic to do so.

```r
library(doMC)
registerDoMC(6)
foreach(i=1:6) %dopar% sqrt(i)
```