---
title: EXPANDS
permalink: /docs/EXPANDS/
---

## Generate the SNV file
```R
snv <- data.frame(chr=maf$Chromosome,
                  startpos = maf$Start_position,
                  endpos = maf$End_position,
                  AF_Tumor = maf$t_alt_count/(maf$t_alt_count + maf$t_ref_count),
                  Tumor_Sample_Barcode = maf$Tumor_Sample_Barcode,
                  PN_B =  0)

snv <- snv[which(!snv$chr %in% c("X", "Y", "MT")), ]
```

## Generate the segmentation file

## Running EXPANDS
```R
library(expands)
```