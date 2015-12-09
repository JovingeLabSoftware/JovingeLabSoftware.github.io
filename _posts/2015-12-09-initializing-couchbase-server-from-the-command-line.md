---
layout: post
category: blog
date: "2015-12-09 18:10 -0500"
author: JLab Skunkworks
tags: 
  - code snippet
  - couchbase
  - shell
published: true
title: Initializing couchbase server from the command line
---

Once couchbase is installed (`dpkg -i couchbase-server_4.1.0-dp-ubuntu14.04_amd64.deb    `), you can set up from the command line as follows.  One corollary of this is that you do not want to do these things prior to creating and image if you want it to be flexible for either node servers or master servers.  In that case, just install coucbase and then make your image, and then do the following from a script after deployment.

Initialize settings:

```Shell
/opt/couchbase/bin/couchbase-cli node-init -c localhost:8091 -u Administrator -p password \
--node-init-data-path=/data/couchbase/172-31-10-84 \
--node-init-index-path=/data/couchbase/172-31-10-84 
```
I could not get this to work without -u -p, even though the password has not been set yet.

Then to start the cluster:
```Shell
/opt/couchbase/bin/couchbase-cli cluster-init -c localhost:8091 \
	--cluster-username=Administrator \
	--cluster-password=password \
	--cluster-port=8091 \
	--cluster-ramsize=10000 \
	--cluster-index-ramsize=2000 \
	--services=data,index,query
```
Instead of starting a new cluster with cluster-init, to add to an existing cluster I believe you would do this (you can also use server-add and then rebalance as separate steps):

```Shell
couchbase-cli rebalance \
    -c [localhost]:8091 \
    --server-add=[host]:8091 \ 
    --server-add-username=[Administrator] \ 
    --server-add-password=[password]
```
