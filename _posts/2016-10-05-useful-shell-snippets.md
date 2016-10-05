---
layout: post
category: blog
date: '2016-10-05 09:09 -0400'
author: JLab Skunkworks
published: false
title: Useful shell snippets
tags:
  - code snippet
  - shell
---
Here is a collection of useful shell (bash) snippets used in the course of my bioinformatic exploits

Look for a sequence in sequencing data (case insensitive)

```
echo CAGCTTCTCTTTAATGTCACGGACAATT | tr [a-z] [A-Z] | grep -f - A10_L00^CR1_001.fastq
```

