---
title: scarHRD (Homologous Recombination Deficiency)
permalink: /docs/scarHRD/
---

To run scarHRD we can use the allele specific copy number calls from either FACETS or ABSOLUTE. For this example, we will be using the FACETS output.

## Generating input file
To generate the input file for scarHRD we need the allele specific copy number calls from the FACETS cncf table, and the tumor ploidy value.

If you're running scarHRD in the same environment as the FACETS run:
```R
library(data.table)

seg <- data.frame(SampleID = "SAMPLE_ID",
                  Chromosome = fit$cncf$chrom,
                  Start_position = fit$cncf$start,
                  End_position = fit$cncf$end,
                  total_cn = fit$cncf$tcn.em,
                  A_cn = fit$cncf$tcn.em - fit$cncf$lcn.em,
                  B_cn = fit$cncf$lcn.em,
                  ploidy = fit$ploidy)

# remove sex chromosomes
seg <- seg[which(seg$Chromosome != 23), ]

# format chromosome column
seg$Chromosome <- as.character(seg$Chromosome)
seg$Chromosome <- paste0("chr", seg$Chromosome)

# write data.table to file because scarHRD takes file path as input
write.table(seg, "~/Downloads/scarHRD_input.txt", row.names = F, sep = '\t', quote = F)
```


If you're running scarHRD on the files we saved from the FACETS output:
```R
library(data.table)

summary.output <- fread("~/Downloads/facets_summary.txt")
seg <- fread("~/Downloads/cncf_table.txt")

# make column for major copy number
seg$mcn <- seg$tcn.em - seg$lcn.em

# only keep necessary columns
seg <- seg[ ,c("sample_id", "chrom", "start", "end", "tcn.em", "mcn", "lcn.em")]

# add ploidy information
seg$ploidy <- summary.output[match(seg$sample_id, summary.output$sample_id), ]$ploidy

# rename columns 
names(seg) <- c("SampleID", "Chromosome", "Start_position", "End_position",
                "total_cn", "A_cn", "B_cn", "ploidy")

# remove sex chromosomes
seg <- seg[which(seg$Chromosome != 23), ]

# format chromosome column
seg$Chromosome <- as.character(seg$Chromosome)
seg$Chromosome <- paste0("chr", seg$Chromosome)

# write data.table to file because scarHRD takes file path as input
write.table(seg, "~/Downloads/scarHRD_input.txt", row.names = F, sep = '\t', quote = F)
```

## Running scarHRD

```R
library(scarHRD)

scarHRD::scar_score("~/Downloads/scarHRD_input.txt",
                    seqz = FALSE,
                    reference = "grch37")
```