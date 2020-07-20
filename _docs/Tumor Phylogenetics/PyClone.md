---
title: PyClone
permalink: /docs/PyClone/
---

## Generating the mutation file

## Running PyClone

https://www.anduril.org/anduril/bundles/all/doc/index.html?c=sequencing.PyClone

```
PyClone run_analysis_pipeline --in_files ${sep =" " mutation_files} \
--tumour_contents ${sep =" " purities} \
--samples ${sep =" " sample_names} \
--density pyclone_beta_binomial \
--num_iters 10000 \
--prior major_copy_number \
--burnin 1000 \
--thin 1 \
--seed 42 \
--mesh_size 101 \
--min_cluster_size 2 \
--working_dir ~/Downloads
```