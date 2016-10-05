---
layout: post
category: blog
date: '2016-10-05 14:41 -0400'
author: JLab Skunkworks
published: false
title: 'What''s up, DOCK?'
tags: ''
---
## A Docking Pipeline

Here are the steps we followed to setup our docking pipeline, with what are hopefully useful pointers along the way.

1. Install Python (e.g. `apt-get install python2.7`)
2. [Install numpy] (e.g. `apt-get install python-numpy`)
3. [Install biopython](http://biopython.org/wiki/Download)

Note that if you do not have root access, you will want to build these from source, following the general approach for configuring for a local install directory I described earlier [in another context](http://jovingelabsoftware.github.io/blog/2016/02/15/make-r-locally-with-cairo/).

4. Install chimera (headless version, which is listed under ["daily builds"](https://www.cgl.ucsf.edu/chimera/download.html#daily), or try the direct link [here](https://www.cgl.ucsf.edu/chimera/cgi-bin/secure/chimera-get.py?file=alpha/chimera-alpha-linux_x86_64_osmesa.bin)).  This is a user friendly installer that will let you specify the installation directory AND create symlinks somewhere in your path.  So it works great whether or not you have root access on your machine.

