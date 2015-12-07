---
layout: post
category: blog
title: "Creating a Custom AMI for StarCluster"
date: "Mon Dec 07 2015 16:30:00 GMT-0500 (EST)"
tags: 
  - cluster
published: true
author: Andrew
---

Used the [Amazon EC2 AMI Locator](http://cloud-images.ubuntu.com/locator/ec2/) to select the base image to use. I stuck with the Ubuntu 14.04 LTS release. I settled on `ami-7b89cc11`.

Followed the directions [here](http://star.mit.edu/cluster/docs/latest/manual/create_new_ami.html) to create a custom AMI to use with StarCluster.

I launched the base image to modify using the following command:

```bash
starcluster start -o -s 1 -b 0.07 -I r3.xlarge -m ami-7b89cc11 imagehost
```

I ran the commands below on the instance to prepare it. Many thanks to [brunogrande](https://github.com/brunogrande/starcluster-ami-config/blob/master/starcluster-ami.json) for figuring all of this out. I certainly didn't have it working after my first attempt...

```bash
apt-get -y update
apt-get -y upgrade
apt-get -y install build-essential git parallel
apt-get install -y nfs-kernel-server nfs-common rpcbind upstart
ln -s /etc/init.d/nfs-kernel-server /etc/init.d/nfs
ln -s /lib/systemd/system/nfs-kernel-server.service /lib/systemd/system/nfs.service
ln -s /lib/init/upstart-job /etc/init.d/portmap
ln -s /lib/init/upstart-job /etc/init.d/portmap-wait
curl -L https://github.com/brunogrande/starcluster-ami-config/blob/master/sge.tar.gz?raw=true | sudo tar -xz -C /opt/
sudo sh -c 'echo deb http://www.duinsoft.nl/pkg debs all > /etc/apt/sources.list.d/duinsoft.list'
sudo apt-key adv --keyserver keys.gnupg.net --recv-keys 0xE18CE6625CB26B26
sudo apt-get -y update
sudo apt-get -y install update-sun-jre
sudo sh -c 'echo deb https://cran.rstudio.com/bin/linux/ubuntu trusty/ > /etc/apt/sources.list.d/cran.list'
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys E084DAB9
sudo apt-get -y update
sudo apt-get -y install r-base r-base-dev
sudo apt-get -y install python-pip
sudo pip install ruffus==2.4.1 pyyaml==3.11 drmaa==0.7.6
sudo pip install -i https://testpypi.python.org/pypi kronos
apt-get install libcurl4-openssl-dev libxml2-dev -y
R -e "install.packages('devtools', repos = 'http://cran.rstudio.com/')"
```

I created a EBS backed AMI with the following command:

```bash
starcluster ebsimage i-e2f84f54 jlab-skunkworks
```

Our resulting AMI is: `ami-f34b0599`.

Below is an example StarCluster template using our custom AMI:

```
[cluster jlabskunk]
EXTENDS = smallcluster
NODE_IMAGE_ID = ami-f34b0599
KEYNAME = my-key-pair
NODE_INSTANCE_TYPE = m3.medium
CLUSTER_SIZE = 3
SPOT_BID = 0.02
```

Which can be launched with:

```bash
starcluster start -c jlabskunk mycluster
```



