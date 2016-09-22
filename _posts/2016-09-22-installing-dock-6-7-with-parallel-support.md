---
layout: post
category: blog
date: '2016-09-22 11:32 -0400'
author: JLab Skunkworks
published: true
title: Installing Dock 6.7 with parallel support
---
Here is how I installed Dock 6.7 in a local directory on our HPC cluster. I already install MPICH locally, and that is needed to support parallel processing in Dock.  [This blog post](http://jovingelabsoftware.github.io/blog/2016/02/15/installing-openmpi-and-rmpi-from-source/) describes installing openMPI, and installing MPICH should install in a similar fashion.  Note that Dock 6 does not support openMPI out of the box (though [they claim](http://dock.compbio.ucsf.edu/DOCK_6/dock6_manual.htm) it should be easy to adapt the build to use openmpi instead of MPICH).

## Obtain source code

- Step 1. [Request License](http://dock.compbio.ucsf.edu/Online_Licensing/dock_license_application.html)
- Step 2. Download and untar


```
wget https://dock.compbio.ucsf.edu/dock.6.7_source.tar.gz --no-check-certificate
tar -zxvf dock.6.7._source.tar.gz
cd dock6/install
export MPICH_HOME=/home/eric.kort/jovinge-primary/local
```

## Create a script to equate yacc to bison
The build script expects yacc, even though we are using a GNU build recipe provided by Dock which one might have assumed would use the GNU equivalent, bison.  So we need to create a little script that passes all yacc calls to bison.  (Thanks to [paxdiablo's answer on stackexhchange](http://stackoverflow.com/questions/10733238/make-yacc-command-not-found-after-installing-bison).  I saved this script to `/home/eric.kort/jovinge-primary/local/bin/yacc`

```
#! /bin/sh
exec '/usr/bin/bison' -y "$@"
```
Then make it executable

```
chmod 754 yacc
```

## Build

Now timet o build

```
make
make install
make test
```

Build completed without errors, however I did see some errors on make test.  Most worrisome is that the `dock` executable could not be found.  After some poking around it turns out that when building with cluster support, the exectuable is named `dock.mpi`.  So let's create a symlink:

```
cd ../bin
ln -s dock6.mpi dock6
```

Now most of the tests pass, though there are still some `possible FAILURE`s.  I am not sure how significant they are, only time will tell.  Notably, though, the ligand sampling tutorial ran fine.

For future reference, something like this should be added to .bashrc to tell `dock` (and the rest of your ecosystem) where the MPICH executables are:

```
export MPICH_HOME=/home/eric.kort/jovinge-primary/local

```
