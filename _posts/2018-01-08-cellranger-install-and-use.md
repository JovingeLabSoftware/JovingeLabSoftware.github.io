---
layout: post
category: blog
date: '2018-01-08 15:26 -0500'
author: JLab Skunkworks
published: false
title: CellRanger install and use
---
## 10XGenomics CellRanger quick start

The first step is to download the cellranger and reference genome tarballs.  These are available from the 10XGenomics [dowload page](https://support.10xgenomics.com/single-cell-gene-expression/software/downloads/latest).  Note that you need to sign in with your email to agree to the EULA, and then a token is generated.  While this precludes putting direct links here, they are nice enough to provide customized `curl` commands to make things easy. 

Also required is Illumina's `bcl2fastq`, which takes a while to build:

```
wget ftp://webdata2:webdata2@ussd-ftp.illumina.com/downloads/software/bcl2fastq/bcl2fastq2-v2-20-0-tar.zip
unzip bcl2fastq2-v2-20-0-tar.zip
tar -zxvf bcl2fastq2-v2.20.0.422-Source.tar.gz  
export INSTALL_DIR=~/local
cd bcl2fastq
./src/configure --prefix=~/local
make
make install

```
Now unarchive the cell ranger suite.

```
tar -zxvf cellranger-2.1.0.tar.gz 
```

And then move the cellranger-2.1.0 directory to somewhere on your path, and/or edit your .bashrc file to add its location to your path.

Move the reference genome data tarball somewhere convenient (I use ~/local/srv/seq/chromium), then unarchive it:

```
tar -zxvf refdata-cellranger-GRCh38-1.2.0.tar.gz
```

Finally, run the test:

```
cellranger testrun --id=tiny

```
