---
title: SomaticSignatures (NMF)
permalink: /docs/SomaticSignatures/
---

```R
# CRAN packages
library(data.table)
library(pheatmap)

# Bioconductor packages
library(TCGAbiolinks)
library(BSgenome.Hsapiens.1000genomes.hs37d5)
library(maftools)
library(SomaticSignatures)
library(SomaticCancerAlterations)
library(VariantAnnotation)
```

```R
query <- GDCquery(project = "TCGA-SKCM", 
                  data.category = "Simple nucleotide variation", 
                  data.type = "Simple somatic mutation",
                  access = "open",
                  file.type = "SKCM_pairs.aggregated.capture.tcga.uuid.automated.somatic.maf",
                  legacy = TRUE)

GDCdownload(query)
maf <- GDCprepare(query)
```

```R
# only SNPs should be used for determining SNV mutational signatures
snp.maf <- maf[which(maf$Variant_Type == "SNP"), ]


# Patient_ID and Sample_ID are required column names for SomaticSignatures
snp.maf$Patient_ID <- snp.maf$Tumor_Sample_Barcode
snp.maf$Sample_ID <- snp.maf$Tumor_Sample_Barcode

# convert MAF to GRange object
GRange <- SomaticCancerAlterations:::.maf2gr(snp.maf, seq_info = NULL)

# generate VRanges object from GRanges object
vr <- VariantAnnotation::VRanges(
  seqnames = seqnames(GRange),
  ranges = ranges(GRange),
  ref = GRange$Reference_Allele,
  alt = GRange$Tumor_Seq_Allele2,
  sampleNames = GRange$Sample_ID,
  seqinfo = seqinfo(GRange)
)

# generate counts for number of mutations in each of the 96 trinucleotide contexts
motifs = SomaticSignatures::mutationContext(vr, BSgenome.Hsapiens.1000genomes.hs37d5)

# normalize the counts by frequency
mm =  SomaticSignatures::motifMatrix(motifs, normalize = TRUE)
```

```R
Run NMF for between 2 to 8 active signatures
n.sigs = 2:8

# Determine residual sum of squares and variance explained
nmf = SomaticSignatures::assessNumberSignatures(mm, n.sigs, nReplicates = 5)

# plot residual sum of squares and variance explained
SomaticSignatures::plotNumberSignatures(nmf)
```

```R
# based on plots, 7 signatures seems optimal
n.sigs = 7

# perform NMF decomposition with 7 signatures 
sigs.nmf = SomaticSignatures::identifySignatures(mm, n.sigs, 
                                                 SomaticSignatures::nmfDecomposition, "brunet")

# extract signatures data.frame
w = sigs.nmf@signatures
```

```R
# read in mutational signatures reference file (COSMIC V2) and format
sigs = data.table::fread(input = system.file('extdata', 'signatures.txt', package = 'maftools'),
                         stringsAsFactors = FALSE, data.table = FALSE)
colnames(sigs) = gsub(pattern = ' ', replacement = '_', x = colnames(sigs))
rownames(sigs) = sigs$Somatic_Mutation_Type

# restrict to just the normalized mutation counts
sigs = sigs[,-c(1:3)]
```

```R
# rename rows ofsignatures data.frame to be consistent with reference
rownames(w) <- rownames(sigs)

# sanity check
sigs = sigs[rownames(w),]
```

```R
# perform cosine similarity between NMF results and reference signatures
coSineMat = c()
  for(i in 1:ncol(w)){
  sig = w[,i]
  coSineMat = rbind(coSineMat, apply(sigs, 2, function(x){
    round(crossprod(sig, x)/sqrt(crossprod(x) * crossprod(sig)), digits = 3)
  }))
}

  rownames(coSineMat) = colnames(w)
```

```R
# determine which known signature has the highest cosine similarity with each NMF signature
for(i in 1:nrow(coSineMat)){
  print(names(which(coSineMat[i,] == max(coSineMat[i,]))))
}
```

```R
# plot the cosine simarity between each NMF signature and each known signature
pheatmap::pheatmap(mat = coSineMat, cluster_rows = FALSE,
                   main = "cosine similarity against validated signatures")
```