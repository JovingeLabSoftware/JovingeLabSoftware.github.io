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

Within that directory, I then prefixed `fasta/genome.fa` with the following entry:

```
>BARCODE
taagctgaaggctgagagggagcgaggaatcaccatcgacatctccctatggaagttccagaccaacaggttcacaatcaccataatcgatgccccggggcacagggacttcGCGATAattaagaacatgatcacgggcacctctcaggcagatgttgctctcctggtggtctctgcggctacaggggaatttgaggccGCGATAggtgtgtccagaaatggccaaacaagggaacacgctcttctggcctacaccatgggggtcaagcaactgatcgtctgcgtgGCGATAaacaaaatggatctgacggaccctccctacagccacaagcggtttgatgaagttgtcaggaatgtgatggtctatctgaaaGCGATAaagattgggtacaacccggctaccatccccttcgtgcctgtgtagggctggacgggagagaatatatcttcgcccagtcaaGCGATAaagatgggttggtttaaaggttggaaggtgaaacagggagcgaggaatcaccatcgacatctccctatggaagttccagaccaacaggttcacaatcaccataatcgatgccccggggcacagggacttcACAGAGattaagaacatgatcacgggcacctctcaggcagatgttgctctcctggtggtctctgcggctacaggggaatttgaggccACAGAGggtgtgtccagaaatggccaaacaagggaacacgctcttctggcctacaccatgggggtcaagcaactgatcgtctgcgtgACAGAGaacaaaatggatctgacggaccctccctacagccacaagcggtttgatgaagttgtcaggaatgtgatggtctatctgaaaACAGAGaagattgggtacaacccggctaccatccccttcgtgcctgtgtagggctggacgggagagaatatatcttcgcccagtcaaACAGAGaagatgggttggtttaaaggttggaaggtgaaactaagctgaaggctgagagggagcgaggaatcaccatcgacatctccctatggaagttccagaccaacaggttcacaatcaccataatcgatgccccggggcacagggacttcTGAGACattaagaacatgatcacgggcacctctcaggcagatgttgctctcctggtggtctctgcggctacaggggaatttgaggccTGAGACggtgtgtccagaaatggccaaacaagggaacacgctcttctggcctacaccatgggggtcaagcaactgatcgtctgcgtgTGAGACaacaaaatggatctgacggaccctccctacagccacaagcggtttgatgaagttgtcaggaatgtgatggtctatctgaaaTGAGACaagattgggtacaacccggctaccatccccttcgtgcctgtgtagggctggacgggagagaatatatcttcgcccagtcaaTGAGACaagatgggttggtttaaaggttggaaggtgaaactaagctgaaggctgagagggagcgaggaatcaccatcgacatctccctatggaagttccagaccaacaggttcacaatcaccataatcgatgccccggggcacagggacttcGTACGTattaagaacatgatcacgggcacctctcaggcagatgttgctctcctggtggtctctgcggctacaggggaatttgaggccGTACGTggtgtgtccagaaatggccaaacaagggaacacgctcttctggcctacaccatgggggtcaagcaactgatcgtctgcgtgGTACGTaacaaaatggatctgacggaccctccctacagccacaagcggtttgatgaagttgtcaggaatgtgatggtctatctgaaaGTACGTaagattgggtacaacccggctaccatccccttcgtgcctgtgtagggctggacgggagagaatatatcttcgcccagtcaaGTACGTaagatgggttggtttaaaggttggaaggtgaaactaagctgaaggctgagagggagcgaggaatcaccatcgacatctccctatggaagttccagaccaacaggttcacaatcaccataatcgatgccccggggcacagggacttcACACTCattaagaacatgatcacgggcacctctcaggcagatgttgctctcctggtggtctctgcggctacaggggaatttgaggccACACTCggtgtgtccagaaatggccaaacaagggaacacgctcttctggcctacaccatgggggtcaagcaactgatcgtctgcgtgACACTCaacaaaatggatctgacggaccctccctacagccacaagcggtttgatgaagttgtcaggaatgtgatggtctatctgaaaACACTCaagattgggtacaacccggctaccatccccttcgtgcctgtgtagggctggacgggagagaatatatcttcgcccagtcaaACACTCaagatgggttggtttaaaggttggaaggtgaaac

```

This creates a synthetic "Chromosome" and contains all my exogenous sequences pasted end to end. I then needed to edit the GTF file to add the features.  I added these features to `genes.gtf`. (When editing large files with emacs, I find it useful to turn off autosave: `M-x auto-save-mode`)

```
BARCODE jovinge exon    1       500     .       +       .       gene_id "ENSG00000000BC1"; gene_version "3"; transcript_id "ENST00000000BC1"; gene_name "BARCODE1"; gene_source "jovinge-lab"; gene_biotype "pseudogene"; havana_gene "OTTHUMG00000000BC1"; havana_gene_version "2";
BARCODE jovinge exon    501     1000    .       +       .       gene_id "ENSG00000000BC2"; gene_version "3"; transcript_id "ENST00000000BC2"; gene_name "BARCODE2"; gene_source "jovinge-lab"; gene_biotype "pseudogene"; havana_gene "OTTHUMG00000000BC2"; havana_gene_version "2";
BARCODE jovinge exon    1001    1500    .       +       .       gene_id "ENSG00000000BC3"; gene_version "3"; transcript_id "ENST00000000BC3"; gene_name "BARCODE3"; gene_source "jovinge-lab"; gene_biotype "pseudogene"; havana_gene "OTTHUMG00000000BC3"; havana_gene_version "2";
BARCODE jovinge exon    1501    2000    .       +       .       gene_id "ENSG00000000BC4"; gene_version "3"; transcript_id "ENST00000000BC4"; gene_name "BARCODE4"; gene_source "jovinge-lab"; gene_biotype "pseudogene"; havana_gene "OTTHUMG00000000BC4"; havana_gene_version "2";
BARCODE jovinge exon    2001    2500    .       +       .       gene_id "ENSG00000000BC5"; gene_version "3"; transcript_id "ENST00000000BC5"; gene_name "BARCODE5"; gene_source "jovinge-lab"; gene_biotype "pseudogene"; havana_gene "OTTHUMG00000000BC5"; havana_gene_version "2";

```
I then moved the hg38 directory already present to hg38_orig, and the existing star diretory to star_orig, and rebuilt the indices

```

#PBS -l walltime=12:00:00
#PBS -l nodes=1:ppn=28
#PBS -l mem=20gb
#PBS -N STAR_INDEX

cd /primary/projects/jovinge/build

/primary/projects/jovinge/local/share/cellranger/cellranger mkref \
   --genome=hg38
   --fasta=fasta/hg38.fa 
   --genes=genes/genes.gtf
   --nthreads=28
   --localmem=50
   --mem=50

```
