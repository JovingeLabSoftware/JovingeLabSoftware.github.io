---
layout: post
category: blog
date: "2015-12-22 15:06 -0500"
author: JLab Skunkworks
tags: null
published: false
title: "What's this?"
---


You've probably seen this (no pun intended) at the top of some javascript functions:

```
var _self = this;
```

This is because the meaning of `this` (the calling context) can change, for example when callbacks call back and promises are fulfilled, by which time `this` has changed.

However, here is a case I ran in today that suprised me:

```
Object.keys(secondary).forEach(function(k) {
      q += "AND " + this._whereStmt(k, secondary[[k]]);
    });
```  
Experienced javascript programmers no doubt can spot that `this` is acutally undefined in that context.  Here or course is the solution:

```
_self = this;
Object.keys(secondary).forEach(function(k) {
      q += "AND " + _self._whereStmt(k, secondary[[k]]);
    });
```  


