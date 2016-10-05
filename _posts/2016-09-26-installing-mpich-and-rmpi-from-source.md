---
layout: post
category: blog
date: '2016-09-26 15:54 -0400'
author: JLab Skunkworks
published: true
title: Installing MPICH and Rmpi from source
---

**NOTE: As of now, I have not been able to get this to work.  Using mpiexec leads to duplication of the same task accross all nodes in the cluster.  If I do not use mpiexec I get errors along the lines of `match_arg (utils/args/args.c:159): unrecognized argument pmi_args`.  So I have gone back to using SOCK based clusters for now. **


If you are working on a cluster and need to install MPICH and Rmpi from source, there are a couple tricks. This is very similar to [installing OpenMPI and Rmpi from source](http://jovingelabsoftware.github.io/blog/2016/02/15/installing-openmpi-and-rmpi-from-source/), but adapted for MPICH.  OpenMPI was giving me headaches on our HPC cluster, so I am moving to MPICH.

Installing MPICH is pretty straight forward:

```
wget http://www.mpich.org/static/downloads/3.2/mpich-3.2.tar.gz
tar -zxvf mpich-3.2.tar.gz
cd mpich-3.2
./configure --prefix=/path/to/your/local
make
make install
```

You can then build the tests as well if desired.  (Note that

```
cd tests/mpi
./configure --prefix=/path/to/your/local
make

```
And get a cup of coffee.  Then you need to let R know where MPICH is and what kind of MPI it is:

```
install.packages("Rmpi", configure.args = 
					  c("--with-Rmpi-include=/primary/projects/jovinge/local/include",
					    "--with-Rmpi-libpath=/primary/projects/jovinge/local/lib",
					    "--with-Rmpi-type=MPICH2",                                                                 "--with-mpi=/primary/projects/jovinge/local"))
```

Then you should be set to go.  Let's adapt an example [from R bloggers]
(http://www.r-bloggers.com/a-very-short-and-unoriginal-introduction-to-snow/) to our purposes:

```
library(snow)
cl <- makeCluster(20, type = "MPI")
x <- sample(0:50, 1000, replace = TRUE)
bs.mean <- function(v) {
   s <- sample(v, length(v), replace = TRUE)
   mean(s)
}
clusterSetupRNG(cl)
clusterCall(cl, bs.mean, x)
```
