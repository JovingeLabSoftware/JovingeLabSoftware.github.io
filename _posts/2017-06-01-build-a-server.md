---
layout: post
category: blog
date: '2017-06-01 15:41 -0400'
author: JLab Skunkworks
published: false
title: Build A Server
---
## A server from scratch

Periodically, the desire arises to setup a fresh server.  The snippets below are the installations we use on top of a base Ubuntu install to create a full stack server focused on bioinformatics:

```
sudo -s
# Latest R from CRAN
echo "deb http://cran.rstudio.com/bin/linux/ubuntu xenial/" | sudo tee -a /etc/apt/sources.list
gpg --keyserver keyserver.ubuntu.com --recv-key E084DAB9
gpg -a --export E084DAB9 | sudo apt-key add -
apt-get update
apt-get install r-base r-base-dev

# nginx
apt-get install nginx

#texlive
apt-get install texlive texlive-latex-extra

# GitLab
apt-get install curl openssh-server ca-certificates postfix

```