---
layout: post
category: blog
title: Easy R-friendly AWS clusters
date: "Sun Dec 06 2015 22:00:00 GMT-0500 (EST)"
tags: 
  - cluster
  - shell
  - r
published: true
author: Andrew
---

Things inevitably go wrong when you work on big datasets. You often develop and test your code on a small random sample of data points. Sadly, despite your statistically sound efforts, this subset rarely reflects the full heterogeneity present in the data set.

We've all been there -- you run some basic tests, check your code, then launch a job that is going to run for a day or so. You check in the next day to find something has gone wrong on some small but significant subset of edge cases. Bummer.

This process is tedious and slow. Speeding it up is of great interest to allow for more rapid iteration in development. Today we'll walk through creating clusters of AWS machines that will allow you to parallelize your jobs so you can fail faster by computing results more quickly. 

The easiest approach I have found uses the excellent [StarCluster](http://star.mit.edu/cluster/docs/latest/overview.html) toolkit. From the [website](http://star.mit.edu/cluster/):

> StarCluster has been designed to automate and simplify the process of building, configuring, and managing clusters of virtual machines on Amazon’s EC2 cloud. StarCluster allows anyone to easily create a cluster computing environment in the cloud suited for distributed and parallel computing applications and systems.

There are some basic configuration requirements to set up StarCluster to use your AWS credentials. Walk through the [Quick Start](http://star.mit.edu/cluster/docs/latest/quickstart.html) guide to get everything initialized.

With your account configured, you are ready to begin launching clusters! StarCluster uses a templating language to allow you to specify certain characteristics of the cluster of machines you'd like to launch -- see [here](http://star.mit.edu/cluster/docs/latest/manual/configuration.html#defining-cluster-templates) for a full breakdown of what can be specified. 
An example template can be see below. Here we specify a 4 machine cluster of `c3.2xlarge` instances (`CLUSTER_SIZE` & `NODE_INSTANCE_TYPE`), set a spot bid price we'd like to pay for the machines (`SPOT_BID`), and specify a configuration script that should be run on each instance when it is initialized (`USERDATA_SCRIPTS`). 

```
[cluster rcluster]
EXTENDS = smallcluster
KEYNAME = my-key-pair
NODE_INSTANCE_TYPE = c3.2xlarge
CLUSTER_SIZE = 4
SPOT_BID = 0.08
USERDATA_SCRIPTS = /Users/borgmaan/projects/aws_config/config.sh
```

StarCluster has some handy tools that allow you scan spot instance prices on EC2 so you can set reasonable bids and save some money. 

```
$ starcluster spothistory c3.2xlarge
StarCluster - (http://star.mit.edu/cluster) (v. 0.95.6)
Software Tools for Academics and Researchers (STAR)
Please submit bug reports to starcluster@mit.edu

>>> Fetching spot history for c3.2xlarge (VPC)
>>> Current price: $0.0720
>>> Max price: $0.0819
>>> Average price: $0.0738
```

The Amazon Machine Images (AMIs) provided by StarCluster are running an outdated version of Ubuntu (13.04). As such, a slight hack of `/etc/apt/sources.list` is required to get the machines to update packages and install software from `apt`. The script below is an example of a script that can be passed to `USERDATA_SCRIPTS`. The following bash script patches `sources.list` and installs `R` along with some supporting packages. 

```bash
#!/usr/bin/env bash

# patch sources.list
sudo sed -i -e 's/archive.ubuntu.com\|security.ubuntu.com/old-releases.ubuntu.com/g' /etc/apt/sources.list
sudo sed -i 's/us-east-1\.ec2\.//g' /etc/apt/sources.list
apt-get update

# some R packages need these
apt-get install libcurl4-openssl-dev libxml2-dev

# install R from source to get a recent version on our outdated ubuntu distro
wget http://cran.r-project.org/src/base/R-3/R-3.1.2.tar.gz
tar xzf R-3.1.2.tar.gz
cd R-3.1.2
./configure --with-readline=no --with-x=no
make
cp -r bin/* /usr/local/bin/

# install the necessary external packages
R -e "install.packages('roxygen2', repos = 'http://cran.rstudio.com/')"
R -e "install.packages(c('Rmpi', 'devtools', 'snow', 'parallelMap', 
                         'dplyr', 'tidyjson', 'rjson'), 
                         repos = 'http://cran.rstudio.com/')"
R -e "devtools::install_github('mlr-org/mlr')"
```

To launch our cluster we issue the following command:

```
starcluster start -c rcluster mycluster
```

In some future posts we'll discuss parallelizing our `R` code to make use of these networked machines. 



