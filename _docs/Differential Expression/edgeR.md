---
title: edgeR
permalink: /docs/edgeR/
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

Creating a differential gene expression (DGE) list object and perform filtering...
```R
# create DGE list object
dge <- DGEList(counts = exp.mat, group = sample.info$group)

# filter lowly expressed genes
keep <- rowSums(cpm(dge)>1) >= 2
dge <- dge[keep, , keep.lib.sizes=FALSE]
```

Normalize...
```R
# Calculate normalization factors to scale the raw library sizes
dge <- calcNormFactors(dge)
```

Set up the design matrix for running the generalized linear model (GLM)...
```R
# Design matrix
design.mat <- model.matrix(~ 0 + dge$samples$group)
colnames(design.mat) <- c("twt", "non_twt")
```

```R
# Estimate dispersion parameter for GLM
dge <- estimateGLMCommonDisp(dge, design.mat)
dge <- estimateGLMTrendedDisp(dge, design.mat, method="power")
dge <- estimateGLMTagwiseDisp(dge,design.mat)
```

```R
# Model fitting
fit.edgeR <- glmFit(dge, design.mat)
```

```R
# Differential expression
contrasts.edgeR <- makeContrasts(twt - non_twt, levels=design.mat)
lrt.edgeR <- glmLRT(fit.edgeR, contrast=contrasts.edgeR)
```

```R
p.threshold <- 0.05

# Access results tables
edgeR_results <- lrt.edgeR$table
sig.edgeR <- decideTestsDGE(lrt.edgeR, adjust.method="BH", p.value = p.threshold)
genes.edgeR <- row.names(edgeR_results)[which(sig.edgeR != 0)]
```