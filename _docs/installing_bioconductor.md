---
title: Installing Bioconductor
permalink: /docs/bioconductor/
---

## What is Bioconductor?
Bioconductor is a repository of biological data and statistical tools specifically geared towards analysis of biological data using the R programming language. Bioconductor packages can be anything from simply hosting genome references or publicly available data, to a series of statistical tools for generating, preprocessing, and analyzing various types of biological data (e.g. genomic, trancsriptomic, methylation, flow cytometry). Essentially, it's CRAN for bioinformatics and computational biology packages.

## Why install Bioconductor?
Chances are that if you're performing analyses on biological data using the R programming language, you'll end up using at least one R package hosted through Bioconductor. For the purpose of this tutorial, we'll be using Bioconductor to access publicly available TCGA data (e.g. somatic mutation calls) and perform downstream analysis on that data (e.g. mutational signature analysis). 

## How to install Bioconductor

Installing Bioconductor is very simple! It only requires 3 lines of code:
```R
if (!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")
BiocManager::install()
```
The first line checks if the BiocManager package is already installed in your R environment. If not, the second line installs the BiocManager package from CRAN. The third line installs Bioconductor and all of its base packages. 

## How to install packages from Bioconductor

To install additional packages from Bioconductor, use the `install` function and provide the name of the desired package(s) in quotes. The first Bioconductor package we want to install is `"TCGAbiolinks"`, which provides functions to access clinical and mutational data from TCGA studies. 
```R
BiocManager::install("TCGAbiolinks")
```

If you plan to follow this entire tutorial you can download all of the required packages upfront using the code below, or you can choose to download packages a few at a time as you jump around the sections.
```R
BiocManager::install(c("TCGAbiolinks", "BSgenome.Hsapiens.1000genomes.hs37d5", "SomaticSignatures", "SomaticCancerAlterations", "VariantAnnotation", "edgeR", "DESeq2"))
```