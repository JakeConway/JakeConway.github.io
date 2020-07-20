---
title: FACETS
permalink: /docs/FACETS/
---

## Generating pileup file

```
```

## Running FACETS
```R
library(facets)

# to get the same results every time set the seed
set.seed(42)

# read in and format the snp-pileup file
rcmat <- readSnpMatrix()

# restrict SNP sites to those within sequencing depth thresholds
# also restrict SNP sites to those that pass het thresholds (being a true het site in sample)
# also make sure there's, on average, 1 snp every snp.nbhd for segmentation (if not add psuedo snp)
xx <- preProcSample(rcmat)

# perform segmentation
oo <- procSample(xx, cval=150)

# determine allelic copy numbers, tumor ploidy, and tumor purity from segmentation info
fit <- emcncf(oo)
```
```R
# plot segmentation info, allelic copy numbers, and cellular fraction of copy number events
plotSample(x = oo, emfit = fit)
```

```R
# QC plot (the closer the points are to the lines, the better calibrated the results)
logRlogORspider(oo$out, oo$dipLogR)
```

## Calculate copy number CCF
When analyzing the **cncf** output from FACETS, most scientists assume that the **cf.em** (cellular fraction) column is the cancer cell fraction (CCF) of the copy number event. However, **cf.em** is the cellular fraction of the entire sample, which includes both tumor and normal components (unless the tumor sample is 100% pure). You should notice that the only segments with a **cf.em** value of 1.0 (100%) are normal ploidy segments. To calculate the CCF of each copy number event, simply divide the **cf.em** column by the **purity** of the sample. In some cases the **cf.em** value may be *slightly* higher than the **purity** value due to noise, which in that case the CCF estimates can be rounded to 1.0 (100%).

```R
# grab all segments with copy number event
cncf.ccf <- fit$cncf[which(fit$cncf$tcn.em != 2 & fit$cncf$lcn.em != 1), ]

# calculate CCF
cncf.ccf$ccf <- sapply(cncf.ccf, function(x) {
  x <- x / fit$purity
  if(x > 1) {
    x <- 1
  }
  return(x)
})
```