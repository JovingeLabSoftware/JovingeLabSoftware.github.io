---
layout: post
category: blog
date: "2015-12-11 09:47 -0500"
author: JLab Skunkworks
tags: 
  - code snippet
  - shell
  - git
published: true
title: I take it all back
---


A friend of mine (ahem) once committed his or her AWS keys to a github repository.  At least it wasn't in a blog post!  Oh wait, it was.

So of course the thing to do is create a new key set and get rid of the old one.

But it raises the question, how can you purge something out of a github repository?  The following will do it:

```
git filter-branch --force --index-filter \
'git rm --cached --ignore-unmatch path/to/fileToRemove.txt' \
--prune-empty --tag-name-filter cat -- --all
git push origin --force --all
```

Of course anyone who has a local copy of the repository or a fork of it will still have your sensitive data.  So this is really only effective if you catch the mistake right away.

So rotate your keys anyway.
