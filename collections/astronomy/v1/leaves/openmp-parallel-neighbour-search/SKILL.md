---
name: openmp-parallel-neighbour-search
description: Use when counting object pairs or triplets in a large astronomical catalogue
  within a separation range, or measuring 2PCF/3PCF, and a brute-force O(N^2) neighbour
  search is too slow.
when_to_use_negative:
- The problem is not a spatial neighbour search but a non-spatial catalogue operation
  (e.g. mass cut-off sub-cataloguing via temp.sub_catalogue with a Var__Mass_ threshold)
- The catalogue is small enough that a direct O(N^2) pairwise loop is already fast
  enough, adding chain-mesh and OpenMP complexity without benefit
- The target environment lacks the required CBL external dependencies (GSL, FFTW,
  OpenMP)
tools:
- name: Cosmo Bologna Lib
  role: Source of the parallel chain-mesh algorithm and clustering classes used to
    count pairs and triplets for 2PCF/3PCF estimation
  repo: github.com/federicomarulli/CosmoBolognaLib
- name: C++
  role: Implementation language for the chain-mesh structure, neighbour-retrieval
    routine and OpenMP-parallel counting loops
- name: GSL
  role: Numerical functions linked against during neighbour retrieval, having fully
    substituted Numerical Recipes in CBL from V3.0
- name: OpenMP
  role: Parallelizes the pair/triplet counting loops over chain-mesh cells
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2016.01.005
    title: 'CosmoBolognaLib: C++ libraries for cosmological calculations'
  source_content_hash: sha256:1780a781c5a0fade10d2c43e550410a03318e407e85f5796c892d65505536520
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - C++
  - Cosmo Bologna Lib
  - GSL
  - OpenMP
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# openmp-parallel-neighbour-search

## Summary

Implements the CosmoBolognaLib chain-mesh spatial search: the catalogue volume is partitioned into cubic cells whose side is set by the largest requested separation scale, and neighbour retrieval visits only relevant cells, with pair/triplet counting loops parallelized via OpenMP for two- and three-point correlation function estimation.

## When to use

Use when counting object pairs or triplets in a large astronomical catalogue within a separation range, or measuring 2PCF/3PCF, and a brute-force O(N^2) neighbour search is too slow.

## When NOT to use

- The problem is not a spatial neighbour search but a non-spatial catalogue operation (e.g. mass cut-off sub-cataloguing via temp.sub_catalogue with a Var__Mass_ threshold)
- The catalogue is small enough that a direct O(N^2) pairwise loop is already fast enough, adding chain-mesh and OpenMP complexity without benefit
- The target environment lacks the required CBL external dependencies (GSL, FFTW, OpenMP)

## Inputs

- Catalogue of extragalactic source objects (e.g. a vector of Galaxy objects from a catalogue file, as used in CBL)
- Requested separation range / largest search scale that fixes the chain-mesh cell side
- Dimensionality choice for the chain (1D, 2D or 3D)

## Outputs

- Chain-mesh structure mapping cubic cells to vectors of object indexes
- Neighbour lists of objects within the requested separation range (e.g. written to chain_mesh_test_output.txt)
- Pair/triplet counts feeding two-point and three-point correlation function estimators

## How to apply

Partition the catalogue volume into cubic cells whose side is set by the largest requested search scale, and store in vectors the indexes of all objects belonging to each cell. Implement the neighbour-retrieval routine so that only cells lying within the requested separation range are visited, linking against GSL numerical functions (Numerical Recipes having been fully substituted by GSL in the library from version 3.0). Parallelize the pair/triplet counting loops with OpenMP, following the parallel chain-mesh algorithm CBL uses to count pairs and triplets for two- and three-point correlation function estimation. Construct chains in 1D, 2D and 3D on a test catalogue, retrieve neighbours within a test scale, and validate the neighbour lists against a brute-force O(N^2) reference search before trusting the fast path.

## Related tools

- **Cosmo Bologna Lib** (Source of the parallel chain-mesh algorithm and clustering classes used to count pairs and triplets for 2PCF/3PCF estimation) — github.com/federicomarulli/CosmoBolognaLib
- **C++** (Implementation language for the chain-mesh structure, neighbour-retrieval routine and OpenMP-parallel counting loops)
- **GSL** (Numerical functions linked against during neighbour retrieval, having fully substituted Numerical Recipes in CBL from V3.0)
- **OpenMP** (Parallelizes the pair/triplet counting loops over chain-mesh cells)

## Examples

```
cd Cosmo_Bologna_Lib/ && make && make libFUNC  # compile CBL libraries, then build and run the chain-mesh test writing chain_mesh_test_output.txt
```

## Evaluation signals

- Chain-mesh neighbour lists for the 1D, 2D and 3D test cases match a brute-force O(N^2) reference search exactly
- Only cells lying within the requested separation range are visited during neighbour retrieval (no full-catalogue scan)
- Clustering class interfaces compile and link against the restructured library with GSL (not Numerical Recipes) functions
- OpenMP-parallel pair counts are identical to serial pair counts on the same catalogue and separation range
- Test program successfully constructs 1D, 2D and 3D chains and writes neighbour lists to chain_mesh_test_output.txt

## Limitations

- The Python wrapper covers only a subset of CBL functions; a full parallel chain-mesh workflow may require C++ rather than Python access
- No fixed version/release identifier was given for the CBL download in the paper (only a .tar archive / first-version GitHub upload), so behaviour may vary across library versions
- Correctness depends on choosing the cell side from the largest requested search scale; an inappropriate scale degrades or breaks the cell-visit logic

## Evidence

- [other] The chain-mesh method divides the catalogue into cubic cells and stores the indexes of all objects in each cell in vectors: [evidence span withheld — anchor status: not_found]
