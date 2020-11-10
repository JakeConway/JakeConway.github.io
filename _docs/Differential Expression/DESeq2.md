---
title: DESeq2
permalink: /docs/DESeq2/
---
After generating a gene by sample expression matrix, we need to create a data.frame with sample-level information which will be used to generate the groups to perform differential expresison on.

```R
# create sample-level data.frame
sample.info <- data.frame(sample_id = XX, group = XX)

# convert group variable to a factor (NOTE: the first level will be the reference group)
sample.info$group <- as.factor(sample.info$group)

# set row names as sample_id
rownames(sample.info) <- as.character(sample.info$sample_id)
```

Some minor formatting...
```R
# make sure all samples in the expression matrix are in the sample-level info data.frame
exp.mat <- exp.mat[ , which(names(exp.mat) %in% sample.info$sample_id)]

# order columns of expression matrix by order of sample IDs in sample-level info data.frame
exp.mat <- exp.mat[ , rownames(sample.info)]

# sanity check
all(rownames(sample.info) == colnames(exp.mat))
```

Although setup is the same, this is where DESeq2 starts to differ from edgeR...

```R
dds <- DESeqDataSetFromMatrix(countData = exp.mat,
                              colData = sample.info,
                              design = ~ cohort)

featureData <- data.frame(gene=rownames(exp.mat))
mcols(dds) <- DataFrame(mcols(dds), featureData)
mcols(dds)

keep <- rowSums(counts(dds)) >= 10
dds <- dds[keep,]

dds$cohort <- relevel(dds$cohort, ref = "non_twt")

dds <- DESeq(dds)
res <- results(dds)
```