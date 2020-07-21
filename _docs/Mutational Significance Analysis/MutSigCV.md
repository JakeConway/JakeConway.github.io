---
title: MutSigCV
permalink: /docs/MutSigCV/
---

The simplest way to run MutSigCV is through [genepattern.org](http://www.genepattern.org/).

At the top of the page click Run > Public Server.

If you haven't already, [register](http://cloud.genepattern.org/gp/pages/registerUser.jsf) for an account on the website.

Once you've logged in:
1. Click Browse Modules
2. Scroll down to mutational significance analysis
3. Click MutSigCV

## Generating the mutation table file

The mutation table file is just a MAF file, so just save the MAF file downloaded through TCGAbiolinks locally.
```R
write.table(maf, "~/Downloads/TCGA_SKCM.maf", row.names = F, sep = '\t', quote = F)
```

## Running MutSigCV
After saving the MAF file locally, upload it to the **mutation table file** input by clicking the Upload File button.

The remaining inputs to MutSigCV include 3 reference files, which are listed below:  
**coverage table file:** ftp://ftp.broadinstitute.org/pub/genepattern/example_files/MutSigCV_1.3/exome_full192.coverage.txt  
**covariates table file:** ftp://ftp.broadinstitute.org/pub/genepattern/example_files/MutSigCV_1.3/gene.covariates.txt  
**mutation type dictionary:** ftp://ftp.broadinstitute.org/pub/genepattern/example_files/MutSigCV_1.3/mutation_type_dictionary_file.txt

Fill out the 2 remaining inputs as follows:  
**output filename base:** TCGA_SKCM  
**genome build:** hg19

Click Run!

## Interpreting the output