---
title: Getting the data
permalink: /docs/getting_data/
---
For most of the tutorials involving analysis downstream of somatic mutation calling, we're going to be using the MAF for the skin cutaneous melanoma cohort. To flow seamlessly with the tutorials, we're going to be downloading and accessing TCGA data through the TCGAbiolinks package from Bioconductor. If you haven't downloaded Bioconductor or the TCGAbiolinks package yet, please follow the steps in Installing Bioconductor.

Loading Bioconductor packages is the same as loading any other R package.
```R
library(TCGAbiolinks)
```

The following code will download and generate the MAF for the TCGA-SKCM cohort.
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

## How do I choose the file.type?
If you follow the online vignettes for downloading TCGA data through TCGAbiolinks, you'll notice that they performed a query step without the **file.type** parameter. For the case of the TCGA-SKCM cohort, there is only 1 MAF file to choose from, so this example is straightforward. However, when downloading somatic mutation data for another TCGA cohort there will likely be multiple MAF files to pick from, and you might be confused on which one to choose.

If we query the TCGA-OV (ovarian cancer) cohort, which we will use when performing mutational signagure analysis with SigMA, you'll notice there are 9 MAF files to pick from. When choosing which MAF file to use from this resource, there are 3 things I consider:
1. The number of samples
2. The sequencing platform
3. The tissue definition

```R
query <- GDCquery(project = "TCGA-OV", 
                  data.category = "Simple nucleotide variation", 
                  data.type = "Simple somatic mutation",
                  access = "open",
                  legacy = TRUE)
```

The sequencing platform and tissue information can be accessed as followed:
```R
query.results <- TCGAbiolinks::getResults(query)

query.results$platform
query.results$tissue.definition
```

The number of samples in each MAF file can be determined using the code below. What you'll notice is some of the MAF files only contain a subset of the samples.
```R
n.samples <- sapply(d$cases, function(x) {
  x <- strsplit(x, ",")[[1]]
  return(length(x))
})

n.samples
```

The sequencing platform is important for several reasons. Besides each platform having their pros and cons, if you plan on integrating TCGA samples into your analysis with another cohort, or use TCGA to validate a finding in your cohort, it is preferable to match up the sequencing technologies. The 4 sequencing platforms used here are **Illimina GA (Genome Analyzer)**, **Illimina HiSeq**, **ABI SOLiD**, and **ABI capillary sequencer**.

**Illumina GA** is Illumina's original sequencing technology, which is no longer being updated.  
**ABI capillary sequencing** is basically old school Sanger sequencing. You'll notice that the number of samples with capillary sequencing are much lower than for the other sequencing technologies.  

**Illumina HiSeq** and **ABI SOLiD** are the two sequencing technologies that you'd most likely choose between if you were just about to sequence new samples. **Illumina HiSeq** is Illumina's main workhorse. Compared to **ABI SOLiD** is has longer read lengths (200-250 bases vs. 35), faster turnaround times (3-10 days vs. 7-14), and produces higher throughput (150-800 gigabases/run vs. 155 gigabases/run). On the other hand, **ABI SOLiD** has upwards of a 99.99% base calling accuracy compared to 99.74 by **Illumina HiSeq**. While that doesn't seem like a huge difference, the number of incorrect base calls adds up given the throughput of these technologies.

* The reason for the variation in read lengths and throughput for **Illumina HiSeq** is because there are multiple **Illumina HiSeq** sequencing technologies (e.g. Illumina HiSeq 2500 vs. Illumina HiSeq 2500 Rapid Run).


## How do I download other data?
To download other data types (e.g. copy number, RNA expression, methylation, clinical data) just changed the **Data.category** and **Data.type** inputs. A list of data categories and data types can be downloaded from: [https://docs.google.com/spreadsheets/d/1f98kFdj9mxVDc1dv4xTZdx8iWgUiDYO-qiFJINvmTZs/export?format=csv&gid=2046985454](https://docs.google.com/spreadsheets/d/1f98kFdj9mxVDc1dv4xTZdx8iWgUiDYO-qiFJINvmTZs/export?format=csv&gid=2046985454)