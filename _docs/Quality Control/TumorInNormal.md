---
title: Tumor-in-normal
permalink: /docs/TumorInNormal/
---

## Prerequisites
Prior to running DeTiN to identify tumor-in-normal contamination, we first need to run 3 pipelines:
1. **MuTect:** to generate information on somatic mutations called in the tumor, which will be used to rescue mutations incorrectly filtered out due to presence of tumor-in-normal
2. **GATK Somatic CNV (GATK3.8):** to generate tangent normalization and segmentation information for the GATK Allelic CNV pipeline
3. **GATK Allelic (GATK4.beta.6):** to identify SNP sites and read counts, and generate allelic copy number calls in format readily compatible with DeTiN