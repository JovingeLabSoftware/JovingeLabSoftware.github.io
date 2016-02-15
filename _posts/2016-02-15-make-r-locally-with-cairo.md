---
layout: post
category: blog
date: "2016-02-15 14:52 -0500"
author: eric
tags: 
  - R
  - shell
published: true
title: "Install R, pixman, and cairo locally"
---


Need to install R locally, and want to be able to render graphics to file without X11 resources available?

Here we install pixman, cairo, and then R...

```
# create some nice directory to hold all your locally built software
mkdir local

wget http://cairographics.org/releases/LATEST-cairo-1.14.6
tar -xvf LATEST-cairo-1.14.6
wget http://cairographics.org/releases/LATEST-pixman-0.34.0
tar -xvf LATEST_pixman-0.34.0

# first build pixman backend for cairo
# adjust version numbers to match what actually downloaded
cd pixman-0.34.0 
./configure --prefix=/full/path/to/local
make
make install

# now tell our environment where stuff is (adjust path as needed)
PKG_CONFIG_PATH=/full/path/to/local/lib/pkgconfig
LD_LIBRARY_PATH=/full/path/to/local/lib
export PKG_CONFIG_PATH LD_LIBRARY_PATH

# now build cairo
# adjust version numbers to match what actually downloaded
cd ../cairo-1.14.6
./configure --prefix=/full/path/to/local
make
make install

# now build R with cairo support
wget https://stat.ethz.ch/R/daily/R-devel.tar.gz
tar -zxvf R-devel.tar.gz
cd R-devel
./configure --prefix=/full/path/to/local --with-cairo
make
install
```

Done.

You can test it with:

```
/full/path/to/local/bin/R
png(file="test.png", type="cairo")
plot(1)
dev.off()
q()
```

Finally, if you haven't already, make life easier by adjusting your path by adding the following line to `~/.bashrc` and restarting your bash session:

```
PATH=$PATH:/full/path/to/local/bin
```
