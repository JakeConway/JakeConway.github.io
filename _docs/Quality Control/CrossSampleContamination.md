---
title: Cross Sample Contamination
permalink: /docs/CrossSampleContamination/
---
During sample preparation, DNA is susceptible to cross-contamination from other sources of DNA. Three of the most common types of contamination are 1) cross-individual contamination, 2) within-individual contaminationm and 3) cross-species contamination. 

[ContEst](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3167057/) can be used to identify cross-individual contamination, while [FACETS](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5027494/) and [DeTiN](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6528031/) can be used to identify within-individual contamination. Specifically, FACETS can be used to detect normal DNA contamination of tumor DNA, commonly referred to as tumor purity, while DeTin can be used to detect tumor DNA contamination of normal DNA, commonly referred to as "tumor-in-normal" contamination. 

## ContEst (GATK 3.7)
To run ConEst, we can run the following command line code:
```
java -Xmx4096m -jar /usr/local/bin/GenomeAnalysisTK.jar \
-T ContEst \
-R ${refFasta} \
-I:eval,genotype ${inputBam} \
-l INFO \
-pf ${popVcf} \
-o ${sampleName}.contamination.txt \
-L ${dbSnpIntervalList} \
-L ${targetsIntervalList} \
-isr INTERSECTION \
--trim_fraction 0.03  \
--beta_threshold 0.05 \
-br ${sampleName}.contamination.txt.base_report.txt \
-mbc 100  \
--min_genotype_depth 30
```
The descriptions of the inputs are as follows:  
**-R** Reference FASTA file for the genome reference used (e.g. Hg19 or Hg38)  
**-I** BAM file that we want cross-individual information on  
**-l**  logging level  
**-pf** Reference file containing population allele frequencies for each SNP in HapMap  
**-o** output file name  
**-L** Interval list over which to operate (dbSnpSix sites)  
**-L** Interval list over which to operate (target intervals used for whole-exome bait sets)  
**-isr**  Approach for merging interval lists (here we're saying only sites included in whole-exome bait set that overlap with dbSNP sites)  
**--beta_threshold** Confidence threshold to determine if the allele fractions at a site disagree with the genotyping array  
**--trim_fraction** At most, what fraction of sites should be trimmed based on the beta threshold  
**-br** base-level information file name  
**-mbc** The minimum number of bases required to determine if there's contamination in a lane  
**--min_genotype_depth**  Minimum sequencing depth for a site to be considered
