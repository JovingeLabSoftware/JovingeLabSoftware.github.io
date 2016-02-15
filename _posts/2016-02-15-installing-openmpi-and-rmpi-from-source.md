---
layout: post
category: blog
date: "2016-02-15 16:18 -0500"
author: eric
tags: 
  - R
  - shell
  - cluster
published: true
title: Installing openmpi and Rmpi from source
---

If you are working on a cluster and need to install openmpi and Rmpi from source, there are a couple tricks.

Installing openmpi is pretty straight forward:

```
wget https://www.open-mpi.org/software/ompi/v1.10/downloads/openmpi-1.10.2.tar.gz
tar -zxvf openmpi-1.10.2.tar.gz
cd openmpi-1.10.2
./configure --prefix=/path/to/your/local
make
make install
```

Then you need to let R know where openmpi is and what kind of MPI it is:

```
install.packages("snow") # if not done already
install.packages("rlecuyer") # if not done already

install.packages("Rmpi", configure.args = 
					  c("--with-Rmpi-include=/path/to/your/local/include",
					    "--with-Rmpi-libpath=/path/to/your/local/lib",
					    "--with-Rmpi-type=OPENMPI",                                                             "--with-mpi=/path/to/your/local/"))
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
