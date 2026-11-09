---
name: cosmological-catalogue-pair-counting
description: Use when measuring two-point or three-point correlation functions from
  real or simulated extragalactic catalogues, where naive O(N^2) pair/triplet enumeration
  over all objects is prohibitively slow.
when_to_use_negative:
- When the catalogue is small enough that a brute-force O(N^2) search is already fast
  enough — the chain-mesh adds cell-partitioning overhead without benefit
- When the analysis requires full Python wrapper coverage of CBL functions — only
  a subset of the CBL currently has a Python wrapper, so use C++ for full functionality
- When counting pairs across arbitrary non-metric separations that cannot be bounded
  by cubic cells sized to the largest requested search scale
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3399
tools:
- name: Cosmo Bologna Lib
  role: Provides the chain-mesh spatial-search structure and parallel pair/triplet
    counting algorithms used for 2PCF and 3PCF estimation
  repo: https://github.com/federicomarulli/CosmoBolognaLib
- name: C++
  role: Language in which the chain-mesh structure, neighbour-retrieval routine and
    pair/triplet counting loops are implemented
  repo: https://github.com/federicomarulli/CosmoBolognaLib
- name: GSL
  role: Numerical functions the neighbour-retrieval and clustering classes link against
    (Numerical Recipes fully substituted by GSL in V3.0)
- name: OpenMP
  role: Parallelizes the pair/triplet counting loops over chain-mesh cells
- name: Doxygen
  role: Generates the associated documentation for the chain-mesh and clustering classes
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2016.01.005
    title: 'CosmoBolognaLib: C++ libraries for cosmological calculations'
  source_content_hash: sha256:9dd056c776415983f4bddc7a3f4e0bb94f37646f28f793f1cfd126346158c3d7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/federicomarulli/CosmoBolognaLib
  tools:
  - C++
  - Cosmo Bologna Lib
  - Doxygen
  - GSL
  - OpenMP
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cosmological-catalogue-pair-counting

## Summary

Fast counting of object pairs (and triplets) in astronomical catalogues using the CosmoBolognaLib parallel chain-mesh algorithm, which partitions the catalogue volume into cubic cells so neighbour searches touch only relevant cells, enabling optimised two-point and three-point correlation function measurement.

## When to use

Use when measuring two-point or three-point correlation functions from real or simulated extragalactic catalogues, where naive O(N^2) pair/triplet enumeration over all objects is prohibitively slow.

## When NOT to use

- When the catalogue is small enough that a brute-force O(N^2) search is already fast enough — the chain-mesh adds cell-partitioning overhead without benefit
- When the analysis requires full Python wrapper coverage of CBL functions — only a subset of the CBL currently has a Python wrapper, so use C++ for full functionality
- When counting pairs across arbitrary non-metric separations that cannot be bounded by cubic cells sized to the largest requested search scale

## Inputs

- Catalogue of extragalactic sources (real or simulated), e.g. galaxy catalogue files such as cat.dat with object coordinates
- Requested separation scale(s) for pair/triplet counting, which set the chain-mesh cell side
- Optionally a Random Catalogue for correlation function estimation
- Compiled CosmoBolognaLib libraries (GSL, FFTW, OpenMP as external dependencies)

## Outputs

- Neighbour lists per object within the requested separation range (e.g. chain_mesh_test_output.txt)
- Pair and triplet counts usable as input to Two Point Correlation and Three Point Correlation classes for 2PCF/3PCF estimation
- Measured two-point and three-point correlation functions in configuration space

## How to apply

Obtain the CosmoBolognaLib C++ source from github.com/federicomarulli/CosmoBolognaLib and locate the spatial-search/clustering classes. Implement or reuse the chain-mesh structure: partition the catalogue volume into cubic cells whose side is set by the largest requested search scale, storing the vector of object indexes belonging to each cell. Retrieve neighbours by visiting only cells lying within the requested separation range, linking against GSL numerical functions (Numerical Recipes having been fully substituted by GSL as of CBL V3.0). Parallelize the pair/triplet counting loops with OpenMP, following the parallel chain-mesh algorithm specifically designed to count pairs and triplets for 2PCF and 3PCF estimation. Validate by comparing chain-mesh neighbour lists against a brute-force O(N^2) reference search in 1D, 2D and 3D cases, and confirm the clustering class interfaces compile against the library (build with 'make', individual libraries with 'make lib*', e.g. 'make libFUNC').

## Related tools

- **Cosmo Bologna Lib** (Provides the chain-mesh spatial-search structure and parallel pair/triplet counting algorithms used for 2PCF and 3PCF estimation) — https://github.com/federicomarulli/CosmoBolognaLib
- **C++** (Language in which the chain-mesh structure, neighbour-retrieval routine and pair/triplet counting loops are implemented) — https://github.com/federicomarulli/CosmoBolognaLib
- **GSL** (Numerical functions the neighbour-retrieval and clustering classes link against (Numerical Recipes fully substituted by GSL in V3.0))
- **OpenMP** (Parallelizes the pair/triplet counting loops over chain-mesh cells)
- **Doxygen** (Generates the associated documentation for the chain-mesh and clustering classes)

## Examples

```
git clone https://github.com/federicomarulli/CosmoBolognaLib.git && cd CosmoBolognaLib && make && make libFUNC
```

## Evaluation signals

- Chain-mesh neighbour lists match a brute-force O(N^2) reference search for the 1D, 2D and 3D chain cases
- Clustering class interfaces compile against the restructured library (e.g. 'make' and 'make libFUNC' succeed)
- Parallel counting with OpenMP produces pair/triplet counts identical to the serial run
- Neighbour retrieval visits only cells within the requested separation range, i.e. cell side equals the largest requested search scale
- 2PCF/3PCF estimated from the counts is reproducible using the Two Point Correlation class example workflow (Catalogue, Random Catalogue, Two Point Correlation class)

## Limitations

- Only a subset of the CBL has a Python wrapper; full coverage is deferred to a future version, so the chain-mech counting must often be driven from C++
- The 2PCF example depends on local input files (cat.dat, random/, pairs/, output/ directories) that are not described or hosted centrally
- No fixed version/release identifier was given for some download routes (a .tar archive at a personal institutional URL); use the GitHub repository, and note V4.1 restructured two-point and three-point correlation classes and Python building
- Cell side is tied to the largest requested search scale; searches at very different scales require rebuilding the chain-mesh

## Evidence

- [methods] chain-mesh method divides the catalogue into cubic cells and stores the indexes of all objects in each cell in vectors, so that to find all the objects close to a given one, the search is performed only on the relevant cells: [evidence span withheld — anchor status: not_found]
- [intro] the CBL provide highly optimised algorithms to measure two-point (2PCF) and three-point correlation functions (3PCF), exploiting a specifically designed parallel chain-mesh algorithm to count pairs: [evidence span withheld — anchor status: not_found]
