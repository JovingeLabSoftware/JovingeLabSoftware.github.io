---
layout: post
category: blog
date: "2015-12-22 15:06 -0500"
author: Eric
tags: 
  - code snippet
  - node.js
published: true
title: "What's this?"
---


You've probably seen this (no pun intended) at the top of some javascript functions:

```
var _self = this;
```

This is because the meaning of `this` (the calling context) can change, for example when callbacks call back and promises are fulfilled, by which time `this` has changed.

However, here is a case I ran in today that suprised me (I have stripped this down to toy functions, but you get the idea):

```
MYOBJECT.prototype.myFunction = function(a) {
  var q;
  Object.keys(a).forEach(function(k) {
        q += "AND " + this.anotherFunction(k);
      });
}

MYOBJECT.prototype.anotherFunction = function(a) {
   return(a);
}
```  
Experienced javascript programmers no doubt can spot that `this` in that context is (roughly speaking) the calling array, not MYOBJECT.    Here or course is the solution:

```
MYOBJECT.prototype.myFunction = function(a) {
  var _self = this;
  var q;
  Object.keys(a).forEach(function(k) {
        q += "AND " + _self.anotherFunction(k);
      });
}

MYOBJECT.prototype.anotherFunction = function(a) {
   return(a);
}

``` 
The meaning of `_self` remains unadulterated because of the scope in which it is defined.

[More info here](http://howtonode.org/what-is-this)


