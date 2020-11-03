---
title: SomaticSignatures (NMF)
permalink: /docs/SomaticSignatures/
---
Load the `data.table` R package for reading in files and the `pheatmap` R package for plotting the output from NMF analysis.
```R
# CRAN packages
library(data.table)
library(pheatmap)
```

Load the Bioconductor packages `TCGAbiolinks` to obtain the data, `BSgenome.Hsapiens.1000genomes.hs37d5` to serve as a genome reference for mutation calls, `maftools` to obtain the 96 trinucleotide signature references, `SomaticSignatures` to perform NMF, `SomaticCancerAlterations` to convert the MAF file data to a GRanges object, and `VariantAnnotation` to convert GRange objects into VRange objects. In short, GRange objects are R objects for storing genomic location information, and VRange objects extend GRange objects to include variant call information (e.g. reference allele, alternate allele, reference allele counts, alternate allele counts, etc.).
```R
# Bioconductor packages
library(TCGAbiolinks)
library(BSgenome.Hsapiens.1000genomes.hs37d5)
library(maftools)
library(SomaticSignatures)
library(SomaticCancerAlterations)
library(VariantAnnotation)
```

Download and generate the MAF for the TCGA-SKCM cohort.
```R
query <- GDCquery(project = "TCGA-OV", 
                  data.category = "Simple nucleotide variation", 
                  data.type = "Simple somatic mutation",
                  access = "open",
                  legacy = TRUE)

GDCdownload(query)
maf <- GDCprepare(query)
```

While indel and double nucleotide polymorphism (DNP) mutational signatures exist, the most common type of mutational signature analyses performed is SNV mutational signatures. Each type of mutational signatures (e.g. SNV, indel, DNP) has a different set of reference signatures with different etiologies. For a comprehensive list of mutational signatures please see: [https://cancer.sanger.ac.uk/cosmic/signatures](https://cancer.sanger.ac.uk/cosmic/signatures). The three sets of mutational signatures in red (SBS, DBS, and ID) are the *new* reference sets of signatures for SNVs, DNPs, and indels **derived and validated in whole-genome sequencing (WGS) data**, but not whole-exome sequencing (WES) data. **Since we are using WES mutation calls, we will be using the reference set of SNV mutational signatures referred to as [mutational signatures version 2](https://cancer.sanger.ac.uk/cosmic/signatures_v2.tt), which were derivied from WES and validated in both WGS and WES.** 
```R
# only SNPs should be used for determining SNV mutational signatures
snp.maf <- maf[which(maf$Variant_Type == "SNP"), ]


# Patient_ID and Sample_ID are required column names
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
```
After converting the SNV mutational calls to a VRanges object, determine the trinucleotide sequence context for each variant using the genome reference and subsequently normalize the frequency of mutations in each trinucleotide conext. The normalization is performed to adjust for the tumor mutational burden, and so that the data is in the same format as the reference set of signatures. If you go back and look at the [reference set of signatures on the COSMIC website](https://cancer.sanger.ac.uk/cosmic/signatures_v2.tt), you will notice the y-axis is in terms of probability (i.e. frequency). 
```R
# generate counts for number of mutations in each of the 96 trinucleotide contexts
motifs = SomaticSignatures::mutationContext(vr, BSgenome.Hsapiens.1000genomes.hs37d5)

# normalize the counts by frequency
mm =  SomaticSignatures::motifMatrix(motifs, normalize = TRUE)
```
Below we will run NMF on the normalize trinucleotide counts. Here we're running NMF to decompose the mutations into between 2 to 8 mutational signatures and performing 5 replicates for each (i.e. 5 replicates when decomposing into 2 signatures, 5 replicates when decomposing into 3 signatures, etc.). The reason we perform replicates is because noise in the mutation calls can affect the stability of factorization and convergence of the algorithm, which is reflected in the variation between the residual sum of squares (RSS), or "fit", of the decomposition. One of the simplest ways to choose the optimal number of signatures is to choose the number of signatures such that increasing the number of signatures by 1 doesn't result in a drastically lower RSS.
```R
# Run NMF for between 2 to 8 active signatures
n.sigs = 2:8

# Determine residual sum of squares and variance explained
nmf = SomaticSignatures::assessNumberSignatures(mm, n.sigs, nReplicates = 5)

# plot residual sum of squares and variance explained
SomaticSignatures::plotNumberSignatures(nmf)
```
Based on the residual RSS plot, 7 signatures appears to be the optimal number of signatures for NMF decomposition.
```R
# based on plots, 7 signatures seems optimal
n.sigs = 7

# perform NMF decomposition with 7 signatures 
sigs.nmf = SomaticSignatures::identifySignatures(mm, n.sigs, 
                                                 SomaticSignatures::nmfDecomposition, "brunet")
```
The result of `SomaticSignatures::identifySignatures` provides us with 2 important matrices:
1. An S (signatures) by N (samples) matrix, which can be interpreted as the relative contribution of each mutational signature to each samples mutational spectrum.  
2. A 96 by S (signatures) matrix, which can be interpreted as the frequency of mutations in each 96 trinucleotide bin for each signature.  
Now that we have this information, we want to figure out which mutational signatures each of our S signatures correspond to. To do this we will perform cosine similarity between our 96 by S matrix, and the set of reference signatures.

```R
# extract signatures data.frame
w = sigs.nmf@signatures

# read in mutational signatures reference file (COSMIC V2) and format
sigs = data.table::fread(input = system.file('extdata', 'signatures.txt', package = 'maftools'),
                         stringsAsFactors = FALSE, data.table = FALSE)
colnames(sigs) = gsub(pattern = ' ', replacement = '_', x = colnames(sigs))
rownames(sigs) = sigs$Somatic_Mutation_Type

# restrict to just the normalized mutation counts
sigs = sigs[,-c(1:3)]

# rename rows ofsignatures data.frame to be consistent with reference
rownames(w) <- rownames(sigs)

# sanity check
sigs = sigs[rownames(w),]

# perform cosine similarity between NMF results and reference signatures
coSineMat = c()
  for(i in 1:ncol(w)){
  sig = w[,i]
  coSineMat = rbind(coSineMat, apply(sigs, 2, function(x){
    round(crossprod(sig, x)/sqrt(crossprod(x) * crossprod(sig)), digits = 3)
  }))
}

  rownames(coSineMat) = colnames(w)

# determine which known signature has the highest cosine similarity with each NMF signature
for(i in 1:nrow(coSineMat)){
  print(names(which(coSineMat[i,] == max(coSineMat[i,]))))
}
```
Now that we know which reference signatures each of our decomposed signatures represents, we can plot the data to show the results! One thing you should notice is that some signature references are very similar to one another, and therefore some of our decomposed signatures may have high cosine similarity to multiple signatures.

```R
# plot the cosine simarity between each NMF signature and each known signature
pheatmap::pheatmap(mat = coSineMat, cluster_rows = FALSE,
                   main = "cosine similarity against validated signatures")
```