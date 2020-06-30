---
title: Installing Bioconductor
permalink: /docs/bioconductor/
---

## What is Bioconductor?

## Why install Bioconductor?

## How to install Bioconductor

Installing Bioconductor is very simple! It only requires 3 lines of code:
```R
if (!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")
BiocManager::install()
```