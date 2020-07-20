---
title: SigMa (Likelihood)
permalink: /docs/SigMa/
---

```R
query <- GDCquery(project = "TCGA-OV", 
                  data.category = "Simple nucleotide variation", 
                  data.type = "Simple somatic mutation",
                  access = "open",
                  file.type = "genome.wustl.edu_OV.IlluminaHiSeq_DNASeq_automated.1.1.0.somatic.maf",
                  legacy = TRUE)

GDCdownload(query)
maf <- GDCprepare(query)
```

```R
library(SigMA)
```

```R
# SigMA requires file paths as input, rather than a data.frame or matrix
write.table(maf, "~/Downloads/TCGA_OV_somatic.maf", row.names = F, sep = '\t', quote = F)
```

```R
# SigMA requires file paths as input, rather than a data.frame or matrix
write.table(maf, "~/Downloads/TCGA_OV_somatic.maf", row.names = F, sep = '\t', quote = F)

maf.file <- "~/Downloads/TCGA_OV_somatic.maf"
```

```R
genomes_matrix <- SigMA:::make_matrix(maf.file, file_type = 'maf')
genomes <- SigMA:::conv_snv_matrix_to_df(genomes_matrix)
```

```R
write.table(genomes,
            "~/Downloads/SigMA_DF.txt",
            sep = ',',
            row.names = F,
            col.names = T ,
            quote = F)
```

```R
SigMA:::run("~/Downloads/SigMA_DF.txt",
            "~/Downloads/SigMA_output.txt",
            data = "seqcap", 
            do_assign = T,
            tumor_type = "ovary",
            do_mva = T, 
            lite_format = T, 
            check_msi = F)
```