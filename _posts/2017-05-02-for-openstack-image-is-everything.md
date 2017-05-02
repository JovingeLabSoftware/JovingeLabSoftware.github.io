---
layout: post
category: blog
date: '2017-05-02 09:25 -0400'
author: JLab Skunkworks
published: true
title: 'For openstack, image is everything'
tags:
  - shell
  - OpenStack
---
### Getting images for openstack deployment

Is your favorite flavor of Linux not available on your openstack deployment?  No problem.  You can use AWS style AMIs as well as disk images with Openstack.  There are a couple of methods:

**Download an iso and create an image.**  

This option assumes you have setup the openstack environment within your shell session.  Doing that is simple...just login to the openstack dashboard, navigate to Compute-->Access & Security-->Api Access.  There is a button there to download a shell script (OpenStack RC File) that you then source within your current session.  This sets the necessary enviornment variables to use the OpenStack command line tools.  Assuming you have saved this file as `~/.openstackrc` you can proceed as follows:

```
cd ~/
. .openstackrc
wget https://cloud-images.ubuntu.com/xenial/current/xenial-server-cloudimg-amd64-disk1.img
openstack image create --disk-format qcow2 \
--file xenial-server-cloudimg-amd64-disk1.img ubuntu_xenial
 
```

(Note that the above assumes the image is a QCOW2 formatted image, as opposed to, say, AMI like Amazon Images or raw format.  Not sure what format your image is?  No problem, try using the file command:

```
file xenial-server-cloudimg-amd64-disk1.img
> build/xenial-server-cloudimg-amd64-disk1.img: QEMU QCOW Image (v2), 2361393152 bytes
```

**Create via the dashboard.**  

Just navigate to Compute-->Images and click "Create Image" and follow the prompts.  Note that you must provide a direct link to the disk image--if the link you have redirects, this will fail.  Also you must know the format for the disk image (likely QCOW2 for Ubuntu cloud images, and AMI for Amazon Images?).

Obviously the dashboard method is easier. The only drawback is that if something goes wrong you may not know why, or even realize something went wrong.  The command line gives more informative feedback.
