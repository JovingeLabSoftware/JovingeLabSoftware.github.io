---
layout: post
category: blog
date: "2015-12-22 10:59 -0500"
author: Eric
tags: 
  - code snippet
  - shell
published: true
title: Branching out
---

Ideally, the git work cycle looks like this:

- Create a branch
- Do your work
- Merge back to master

This keeps developers from stepping on eachothers toes and allows prolonged efforts on major features.  Here are some handy commands for actually doing it:

Assuming you have alread git clone'd a repository, you can create a new branch like so:

```
git checkout -b mynewbranch
```

Alternatively, to clone an existing branch

```
git clone -b mynewbranch https://github.com/YourSoftware/YourRepo.git
```
Then make your changes, and push to the remote branch

```
git commit -am "message"
git push origin mynewbranch
```

Finally, when you are ready to push your changes back to the master you can:

```
git checkout -b master
git merge mynewbranch
#(then git push???)
```
If all goes well (or if it doesn't but you resolve the conflicts), you can remove the branch

```
git -d mynewbranch
```

