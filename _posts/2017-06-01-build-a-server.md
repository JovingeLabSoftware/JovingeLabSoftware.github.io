---
layout: post
category: blog
date: '2017-06-01 15:41 -0400'
author: JLab Skunkworks
published: false
title: Build A Server
---
## A server from scratch

Periodically, the desire arises to setup a fresh server.  The snippets below are the installations we use on top of a base Ubuntu install to create a full stack server focused on bioinformatics.  This could be rolled into a post-deployment script for OpenStack, EC2, etc.

```
sudo -s

# Latest R from CRAN
echo "deb http://cran.rstudio.com/bin/linux/ubuntu xenial/" | sudo tee -a /etc/apt/sources.list
gpg --keyserver keyserver.ubuntu.com --recv-key E084DAB9
gpg -a --export E084DAB9 | sudo apt-key add -
apt-get update
apt-get install -y r-base r-base-dev

# R studio
apt-get install gdebi-core
wget https://download2.rstudio.org/rstudio-server-1.0.143-amd64.deb
gdebi rstudio-server-1.0.143-amd64.deb

# nginx
apt-get install -y nginx

#texlive
apt-get install -y texlive texlive-latex-extra

# GitLab
debconf-set-selections <<< "postfix postfix/mailname string your.hostname.com"
debconf-set-selections <<< "postfix postfix/main_mailer_type string 'Internet Site'"
apt-get install -y postfixapt-get install curl openssh-server ca-certificates postfix 

# shell script based install...not for the faint of heart.  But this is a fresh server so ok.
curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
apt-get install -y gitlab-ce
gitlab-ctl reconfigure
```

Then we need to configure GitLab to use our top level NGINX proxy server (so we can run other apps too, like RStudio Server).  Based on yokodev's answer to SO question 24090624, edit two lines of /etc/gitlab/gitlab.rb:

```
external_url 'http://10.152.222.18/gitlab'
nginx['listen_port'] = 8081
```
Then, from the command line:

```
nginx-ctl reconfigure
nginx-ctl restart

```

Then we need to add the rstudio endpoint to our top level nginx.  Add this to /etc/nginx/sites-available/default:

```
server {
 # a bunch of stuff will already be there, then add this:
 location /rstudio/ {
	rewrite ^/rstudio/(.*)$ /$1 break;
 	proxy_pass http://localhost:8787;
 	proxy_redirect http://localhost:8787/ $scheme://$host/rstudio/;
 	proxy_http_version 1.1;
 	proxy_set_header Upgrade $http_upgrade;
 	proxy_set_header Connection $connection_upgrade;
 	proxy_read_timeout 20d;
 }
 
} # <-- this closing bracket will already be there
```
Similarly, add this within the http directive in /etc/nginx/nginx.conf:

```
http {
 # a bunch of stuff is already there, then add this

 map $http_upgrade $connection_upgrade {
      default upgrade;
      ''      close;
    }
    
} # <-- this closing bracket will already be there
```