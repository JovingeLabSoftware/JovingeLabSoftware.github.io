---
layout: post
category: blog
date: '2018-01-24 10:00 -0500'
author: JLab Skunkworks
published: false
title: Add gene sequences to STAR index
tags: ''
---
In the course of our analysis of data coming out of the 10XGenomics Chromium pipeline, the need arises to add custom gene sequences to the STAR index for alignment and feature counting.  (Why, you ask? Stay tuned!)

First we need to download the 10X build of the reference genome if you have not already done that as part of your cellranger installation (as described [here](http://jovingelabsoftware.github.io/blog/2018/01/08/cellranger-install-and-use/):

```
curl -O http://cf.10xgenomics.com/supp/cell-exp/refdata-cellranger-GRCh38-1.2.0.tar.gz
tar -zxvf refdata-cellranger-GRCh38-1.2.0.tar.gz
```

I then created a fasta file for each exogenous construct.  I also edited the GTF file to add these features.  Then I re-indexed genome with STAR.  This was performed on our HPC cluster with the following PBS:

#PBS -l walltime=12:00:00
#PBS -l nodes=1:ppn=28
#PBS -l mem=50gb
#PBS -N STAR_INDEX

cd /projects/primary/jovinge.lab/build

STAR   --genomeFastaFiles genome.fa BC1.fa BC2.fa BC3.fa BC4.fa BC5.fa  \
       --runMode genomeGenerate \
       --genomeDir ./  \
       --sjdbGTFfile genes.gtf \
       --sjdbOverhang 100  \
       --runThreadN 28  

