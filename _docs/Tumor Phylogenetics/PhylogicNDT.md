---
title: PhylogicNDT
permalink: /docs/PhylogicNDT/
---

## PhylogicNDT Modules
PhylogicNDT has 6 modules that can be used to analyze tumor evolution:
1. Cluster
2. BuildTree
3. GrowthKinetics
4. SinglePatientTiming
5. LeagueModel
6. PhylogicSim

To determine the phylogenetic relationship between tumor subclones, we only need to use the Cluster and BuildTree modules.

## Cluster
When running the Cluster module, I prefer to use the sif file option. The command line code is much cleaner, and the format provides a much more organized way to keep track of your input files and values.

Sif files have 4 required columns, and 1 optional column:

**sample_id:** the ID of each sample from the patient  
**maf_fn:** a truncated version of the MAF file with additional CCF information  
**seg_fn (optional):** a truncated version of allelic seg calls (from FACETS or ABSOLUTE)  
**purity:** the purity of each tumor sample from the patient (from FACETS or ABSOLUTE)  
**timepoint:** if temporal ordered by biopsy time, if spatial then arbitrary

##### Generating the maf_fn file

##### Generating the sif file

Only a patient ID and the sif file are required to run the Cluster module:
```
./PhylogicNDT.py Cluster -i Patient_ID  -sif Patient.sif
```


## BuildTree
The BuildTree inputs are just a subset of the outputs from the Cluster module:
```
./PhylogicNDT.py BuildTree -i Patient_ID -sif Patient.sif  -m mutation_ccf_file -c cluster_ccf_file 
```

Again, we need to proide the patient ID and the sif file. A description of the mutation CCF file and cluster CFF file output by the cluster module are described below.

**mutation_ccf_file:** This file contains the cluster assignment of each mutation in the MAF in the **maf_fn** file.  
**cluster_ccf_file:** This file contains CCF information on each cluster, such as the average CCF of mutations in the cluster and the probability distribution of the cluster CCF.
