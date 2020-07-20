---
title: Sequencing Coverage
permalink: /docs/SequencingCoverage/
---

## DepthOfCoverage (GATK3.7)

```
java -Xmx15g -jar /usr/GenomeAnalysisTK.jar \
-R ${refFasta} \
-T DepthOfCoverage \
-o ${sampleName} \
-omitBaseOutput \
-pt sample \
-geneList ${geneList} \
-I $bamFile \
-L ${intervalList} \
--minBaseQuality ${minBaseQuality} \
--minMappingQuality ${minMappingQuality}
```


