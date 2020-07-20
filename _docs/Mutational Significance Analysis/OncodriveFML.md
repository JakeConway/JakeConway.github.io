---
title: OncodriveFML
permalink: /docs/OncodriveFML/
---

The simplest way to run OncodriveFML is through the Barcelona Biomedical Genomics (BBG) Lab [website](http://bbglab.irbbarcelona.org/oncodrivefml/home). If you haven't already, sign up for an account on the website using the login button at the top right of the page. 

After you've logged in, click on the [Analysis](http://bbglab.irbbarcelona.org/oncodrivefml/analysis) tab at the top of the page.

Prior to running the analysis we need to do a bit of formatting on the MAF file.

## Generating the Mutations file

The Mutations file is simply just a trimmed down version of the MAF.

```R
oncodrive.input <- maf[ ,c("Chromosome",
                           "Start_position",
                           "Reference_Allele",
                           "Tumor_Seq_Allele2",
                           "Tumor_Sample_Barcode")]

names(oncodrive.input) <- c("CHROMOSOME", 
                            "POSITION",
                            "REF",
                            "ALT",
                            "SAMPLE")

write.table(oncodrive.input, "~/Downloads/oncodrive_input.txt",
            row.names = F, sep = '\t', quote = F)
```

## Filling out the Analysis tab

Back in the [Analysis](http://bbglab.irbbarcelona.org/oncodrivefml/analysis) tab, fill the inputs with the following:

**Analysis title:** TCGA_SKCM  
**Mutations file:** upload ~/Downloads/oncodrive_input.txt here  
**Sequencing:** Whole exome  
**Indels:** Included  
**Genomic elements:** Coding sequence  

Leave the Advanced options inputs as the defaults.

Once all of the inputs are filled out, click the Submit button!

## Interpreting the output