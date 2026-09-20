---
name: cplusplus-class-implementation-for-clustering
description: Use when counting object pairs or triplets in large catalogues for 2PCF/3PCF
  estimation and brute-force O(N^2) searches are too slow; spatial neighbour queries
  must be restricted to relevant cells.
when_to_use_negative:
- The task only needs a one-off neighbour query on a small catalogue where a direct
  O(N^2) search is sufficient — the chain-mesh build overhead is not justified
- The goal is modelling or fitting correlation functions rather than counting pairs/triplets
  from catalogues (CBL provides separate modelling classes for that)
- The input is not a spatial point catalogue but e.g. a cosmological parameter or
  likelihood analysis problem outside CBL's clustering classes
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0121
tools:
- name: Cosmo Bologna Lib
  role: Reference implementation of the parallel chain-mesh algorithm for counting
    pairs and triplets; source to locate spatial-search/clustering classes in
  repo: github.com/federicomarulli/CosmoBolognaLib
- name: C++
  role: Implementation language for the chain-mesh structure and clustering class
    interfaces
- name: GSL
  role: External numerical library linked by the restructured CBL code (Numerical
    Recipes fully substituted by GSL functions since V3.0)
- name: OpenMP
  role: Parallelization of the pair/triplet counting loops in the chain-mesh algorithm
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2016.01.005
    title: 'CosmoBolognaLib: C++ libraries for cosmological calculations'
  source_content_hash: sha256:bb306976319b33b5a672ac7348e512b1f5369bb41273338f3b6738cb6b39e98b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - C++
  - Cosmo Bologna Lib
  - GSL
  - OpenMP
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cplusplus-class-implementation-for-clustering

## Summary

Implement C++ clustering classes in the CosmoBolognaLib style: a chain-mesh spatial index (cubic cells storing object indexes in vectors) plus OpenMP-parallel pair/triplet counting routines for two- and three-point correlation function estimation, using GSL instead of Numerical Recipes.

## When to use

Use when counting object pairs or triplets in large catalogues for 2PCF/3PCF estimation and brute-force O(N^2) searches are too slow; spatial neighbour queries must be restricted to relevant cells.

## When NOT to use

- The task only needs a one-off neighbour query on a small catalogue where a direct O(N^2) search is sufficient — the chain-mesh build overhead is not justified
- The goal is modelling or fitting correlation functions rather than counting pairs/triplets from catalogues (CBL provides separate modelling classes for that)
- The input is not a spatial point catalogue but e.g. a cosmological parameter or likelihood analysis problem outside CBL's clustering classes

## Inputs

- Astronomical catalogue of objects (real or simulated), e.g. galaxy catalogues with positions for clustering measurement
- Requested separation scale(s) defining the chain-mesh cell side and neighbour-search range
- CosmoBolognaLib C++ source tree with Example codes as reference interfaces

## Outputs

- Chain-mesh structure mapping cubic cells to vectors of object indexes (1D, 2D and 3D chains)
- Neighbour lists of objects within the requested separation range (e.g. chain_mesh_test_output.txt)
- Parallel pair/triplet counts usable for two-point and three-point correlation function estimation

## How to apply

Obtain the CosmoBolognaLib C++ source from github.com/federicomarulli/CosmoBolognaLib and locate the spatial-search/clustering classes. Implement the chain-mesh by partitioning the catalogue volume into cubic cells whose side is set by the largest requested search scale, storing the list of object indexes belonging to each cell in vectors; the neighbour-retrieval routine then visits only cells lying within the requested separation range. Link against GSL numerical functions (Numerical Recipes having been fully substituted by GSL since V3.0) and install GSL, FFTW and OpenMP as prerequisites. Parallelize the pair/triplet counting loops with OpenMP, following CBL's specifically designed parallel chain-mesh algorithm for two- and three-point correlation functions. Validate by constructing 1D, 2D and 3D chains on a test catalogue, retrieving neighbours within a test scale, and comparing the neighbour lists against a brute-force O(N^2) reference search.

## Related tools

- **Cosmo Bologna Lib** (Reference implementation of the parallel chain-mesh algorithm for counting pairs and triplets; source to locate spatial-search/clustering classes in) — github.com/federicomarulli/CosmoBolognaLib
- **C++** (Implementation language for the chain-mesh structure and clustering class interfaces)
- **GSL** (External numerical library linked by the restructured CBL code (Numerical Recipes fully substituted by GSL functions since V3.0))
- **OpenMP** (Parallelization of the pair/triplet counting loops in the chain-mesh algorithm)

## Examples

```
cd CosmoBolognaLib/ && make libFUNC && make python
```

## Evaluation signals

- Chain-mesh neighbour lists for 1D, 2D and 3D test chains exactly match a brute-force O(N^2) reference search at the test scale
- The clustering class interfaces compile cleanly against the restructured library (e.g. 'make lib*' builds the target .so, such as libFUNC.so)
- Neighbour retrieval visits only cells lying within the requested separation range, i.e. no cells outside the scale are searched
- OpenMP-parallel pair/triplet counts agree with serial counts for the same catalogue and separation range
- Test program writes neighbour lists to chain_mesh_test_output.txt with expected object indexes for each query

## Limitations

- The chain-mesh cell side is fixed by the largest requested search scale, so very large search radii degrade the cell-granularity advantage over brute force
- Examples in the library depend on local input files (e.g. cat.dat, random/, pairs/, output/ directories) that are not described or hosted with the paper
- No fixed version/release identifier was given for the CBL download in the paper; only a .tar archive at a personal institutional URL (the GitHub repo has no changelog)

## Evidence

- [intro] parallel chain-mesh algorithm to count pairs: [evidence span withheld — anchor status: not_found]
