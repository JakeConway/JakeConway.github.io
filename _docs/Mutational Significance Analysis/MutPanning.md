---
title: MutPanning
permalink: /docs/MutPanning/
---

The simplest way to run MutPanning is through [genepattern.org](http://www.genepattern.org/).

At the top of the page click Run > Public Server.

If you haven't already, [register](http://cloud.genepattern.org/gp/pages/registerUser.jsf) for an account on the website.

Once you've logged in:
1. Click Browse Modules
2. Scroll down to mutational significance analysis
3. Click MutPanning

## Generating the mutation file
```R
mutpanning.input <- maf[ ,c("Hugo_Symbol",
                            "Chromosome",
                            "Start_position",
                            "End_position",
                            "Strand",
                            "Variant_Classification",
                            "Variant_Type",
                            "Reference_Allele",
                            "Tumor_Seq_Allele1",
                            "Tumor_Seq_Allele2",
                            "Tumor_Sample_Barcode")]

names(mutpanning.input)[3:4] <- c("Start_Position", "End_Position")

write.table(mutpanning.input, "~/Downloads/mutpanning_input.txt",
            row.names = F, sep = '\t', quote = F)
```

## Generating the sample annotation file
```R
sample.annot.input <- data.frame(Sample=unique(maf$Tumor_Sample_Barcode),
                                 Cohort="TCGA-SKCM")

write.table(sample.annot.input, "~/Downloads/sample_annotation_input.txt",
            row.names = F, sep = '\t', quote = F)
```

## Running MutPanning
After generating the mutation file and the sample annotation file, upload the files to their respective inputs by either dragging and dropping, or clicking the Upload File button.

Leave the rest of the parameters as the default values.

Click Run!

## Interpreting the output