---
title: deconstructSigs (NNLS)
permalink: /docs/deconstructSigs/
---
One major difference between NNLS and NMF is that NNLS can work in individual samples rather than requiring an entire cohort. However, depending on the number of mutations per sample, NMF may be powered enough to provide insight on individual samples. With whole-exome sequencing data, this is likely not the case. 

deconstructSigs is an R package that can be downloaded through CRAN.
```R
library(deconstructSigs)
library(plyr)
```

In whole-exome sequencing data, we mostly care about SNV mutational signatures. Even if we were powered enough for indel or dinucleotide mutational signatures (i.e. enough indel or dinucleotide mutations), we would still run each separately. This is because the reference signatures (relative frequency of mutation in trinucleotide context for each mutational process) were determined for SNVs, indels and DNPs separately. 
```R
# only SNVs should be used for determining SNV mutational signatures
snp.maf <- maf[which(maf$Variant_Type == "SNP"), ]
```

deconstructSigs (NNLS) requires the same information as SomaticSignatures (NMF).
```R
deconstructSigs.df <- snp.maf[ ,c("Tumor_Sample_Barcode",
                                  "Chromosome",
                                  "Start_position",
                                  "Reference_Allele",
                                  "Tumor_Seq_Allele2")]
deconstructSigs.df$Chromosome <- paste0("chr", deconstructSigs.df$Chromosome)

# requires chrX and chrY to be encoded as chr23 and chr24 respectively (to match reference)
deconstructSigs.df[which(deconstructSigs.df$Chromosome == "chrX"), ]$Chromosome <- "chr23"
deconstructSigs.df[which(deconstructSigs.df$Chromosome == "chrY"), ]$Chromosome <- "chr24"
deconstructSigs.df <- as.data.frame(deconstructSigs.df)
```

Just like the NMF based method, the required input is the frequency of mutations in each trinucleotide context. However, this time it is the frequency in each sample rather than the frequency across the cohort. 
```R
# deconstructSigs signatures
sigs.input <- deconstructSigs::mut.to.sigs.input(deconstructSigs.df,
                                                 sample.id = "Tumor_Sample_Barcode", 
                                                 chr = "Chromosome", 
                                                 pos = "Start_position", 
                                                 ref = "Reference_Allele", 
                                                 alt = "Tumor_Seq_Allele2")
```

Since NNLS works at the individual sample level, **whichSignatures** only runs on one sample at a time. Thus, we use the **lapply** function to loop through the trinucleotide context frequencies of each sample and determine both 1) which mutational signatures are active, and 2) the reltive contribution of each active mutational signature to the mutational spectrum of the tumor.

**deconstructSigs** recommends a relative signature contribution cutoff of 0.06 (6%), which through the authors in-silico analysis, led to a FDR of 1%. As a result, if there are any signatures that have a relative contribution of less than 0.06 via the NNLS model, **deconstructSigs** will set the relative contribution to 0, causing the sum of the relative contributions to sum to less than 1 (100%). Thus, we also havev to rescale the relative contributions of each signature such that the sum of the relative signature contributions equals 1 (100%). 
```R
sig.contrib.list <- lapply(row.names(sigs.input), function(x) {
  row.df <- deconstructSigs::whichSignatures(sigs.input, x, contexts.needed = TRUE, 
                                             signatures.ref = signatures.cosmic)$weights
  row.sum <- rowSums(row.df)
  row.df <- row.df/row.sum
  rownames(row.df) <- x
  return(as.data.frame(row.df))
})
```

After determining the relative contribution of each mutational signature in each tumor, we can merge the results into one data.frame for easier downstream analysis. 
```R
deconstructSigs.results <- plyr::rbind.fill(sig.contrib.list)
rownames(deconstructSigs.results) <- rownames(sigs.input)
```

To provide a comparisons to the NMF method (SomaticSignatures), we can estimate the activity of each mutational signature across by summing the relative contributions for each mutational signature. A more appropriate way to compare the NNLS results with the NMF results would be to multiply the relative contributions of each mutational signature by their respective sample's tumor mutational burden (TMB), and then summing the signature activities across the cohort.
```R
process.ranks <- colSums(deconstructSigs.results)
process.ranks <- sort(process.ranks, decreasing = TRUE)
process.ranks
```