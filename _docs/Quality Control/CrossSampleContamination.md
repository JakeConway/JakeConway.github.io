---
title: Cross Sample Contamination
permalink: /docs/CrossSampleContamination/
---

## ContEst (GATK 3.7)

```
java -Xmx4096m -jar /usr/local/bin/GenomeAnalysisTK.jar \
-T ContEst \
-R ${refFasta} \
-I:eval,genotype ${inputBam} \
-l INFO \
-pf ${popVcf} \
-o ${sampleName}.contamination.txt \
-L ${dbSnpSixIntervalList} \
-L ${targetsIntervalList} \
-isr INTERSECTION \
--trim_fraction 0.03  \
--beta_threshold 0.05 \
-br ${sampleName}.contamination.txt.base_report.txt \
-mbc 100  \
--min_genotype_depth 30  \
--min_genotype_ratio 0.8
```
