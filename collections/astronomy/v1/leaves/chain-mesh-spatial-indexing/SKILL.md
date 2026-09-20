---
name: chain-mesh-spatial-indexing
description: Use when a catalogue of extragalactic objects must be searched for neighbours
  within a separation range to count pairs/triplets for 2PCF/3PCF estimation, and
  a brute-force O(N^2) scan is too slow.
when_to_use_negative:
- The catalogue is small enough that a direct brute-force O(N^2) pair search is already
  affordable
- The task needs only a subset of CBL functionality exposed via Python — the Python
  wrapper covers only a subset of CBL functions, so C++ work is required
- The analysis does not involve spatial neighbour retrieval (e.g. purely angular or
  non-catalogue-based cosmological calculations)
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: Cosmo Bologna Lib
  role: Source library providing the chain-mesh structure, neighbour-retrieval routine
    and parallel pair/triplet counting algorithm to reimplement or reuse
  repo: https://github.com/federicomarulli/CosmoBolognaLib
- name: C++
  role: Implementation language for the chain-mesh structure and clustering class
    interfaces
- name: GSL
  role: Numerical functions linked against by the neighbour-retrieval routine (substituted
    for Numerical Recipes in V3.0)
- name: OpenMP
  role: Parallelization of the pair/triplet counting loops in the chain-mesh algorithm
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2016.01.005
    title: 'CosmoBolognaLib: C++ libraries for cosmological calculations'
  source_content_hash: sha256:8e4794c1dedd93176c179bceb65377a3f13fa429fcc2e54c7d1da807f8e4f090
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/federicomarulli/CosmoBolognaLib
  tools:
  - C++
  - Cosmo Bologna Lib
  - GSL
  - OpenMP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# chain-mesh-spatial-indexing

## Summary

Reconstructs the CosmoBolognaLib chain-mesh spatial-indexing method: partition a catalogue into cubic cells and store object indexes per cell so neighbour searches for pair and triplet counting visit only relevant cells. It underpins the parallel OpenMP counting used for two- and three-point correlation functions.

## When to use

Use when a catalogue of extragalactic objects must be searched for neighbours within a separation range to count pairs/triplets for 2PCF/3PCF estimation, and a brute-force O(N^2) scan is too slow.

## When NOT to use

- The catalogue is small enough that a direct brute-force O(N^2) pair search is already affordable
- The task needs only a subset of CBL functionality exposed via Python — the Python wrapper covers only a subset of CBL functions, so C++ work is required
- The analysis does not involve spatial neighbour retrieval (e.g. purely angular or non-catalogue-based cosmological calculations)

## Inputs

- Catalogue of extragalactic objects (real or simulated) as C++ vectors of object coordinates
- Requested separation range / largest search scale (sets the cubic cell side)
- Dimensionality of the chain: 1D, 2D or 3D

## Outputs

- Chain-mesh structure mapping cubic cells to vectors of object indexes
- Neighbour lists of objects close to each object within the test scale
- Pair and triplet counts for two- and three-point correlation function estimation
- Test output written to chain_mesh_test_output.txt

## How to apply

Obtain the CosmoBolognaLib C++ source from GitHub and locate the spatial-search/clustering classes, then implement the chain-mesh by partitioning the catalogue volume into cubic cells whose side is set by the largest requested search scale, storing the list of object indexes belonging to each cell in vectors. Implement the neighbour-retrieval routine so that only cells lying within the requested separation range are visited, linking against GSL numerical functions (Numerical Recipes having been fully substituted by GSL as of V3.0). Parallelize the pair/triplet counting loops with OpenMP, following the parallel chain-mesh algorithm designed for two- and three-point correlation function estimation in CBL. Validate by constructing 1D, 2D and 3D chains on a test catalogue, retrieving neighbours within a test scale, and checking the neighbour lists against a brute-force O(N^2) reference search.

## Related tools

- **Cosmo Bologna Lib** (Source library providing the chain-mesh structure, neighbour-retrieval routine and parallel pair/triplet counting algorithm to reimplement or reuse) — https://github.com/federicomarulli/CosmoBolognaLib
- **C++** (Implementation language for the chain-mesh structure and clustering class interfaces)
- **GSL** (Numerical functions linked against by the neighbour-retrieval routine (substituted for Numerical Recipes in V3.0))
- **OpenMP** (Parallelization of the pair/triplet counting loops in the chain-mesh algorithm)

## Examples

```
cd Cosmo_Bologna_Lib/ && make
```

## Evaluation signals

- Chain-mesh neighbour lists match a brute-force O(N^2) reference search for the 1D, 2D and 3D test cases
- Only cells lying within the requested separation range are visited during neighbour retrieval
- Cubic cell side equals the largest requested search scale and each cell stores the list of object indexes belonging to it
- Clustering class interfaces compile against the restructured library (e.g. via 'make lib*')
- Test program writes neighbour lists to chain_mesh_test_output.txt for the 1D, 2D and 3D chains

## Limitations

- Only a subset of the CBL has a Python wrapper; full coverage is deferred to a future version, so chain-mesh work must be done in C++
- No fixed version/release identifier is given for the CBL download; only a .tar archive and the GitHub repository
- V3.0 substituted Numerical Recipes with GSL, so reproducing pre-V3.0 code requires the Numerical Recipes dependency
- Examples depend on local input files (e.g. cat.dat, random/, pairs/, output/ directories) that are not described or hosted

## Evidence

- [intro] the CBL provide highly optimised algorithms to measure two-point (2PCF) and three-point correlation functions (3PCF), exploiting a specifically designed parallel chain-mesh algorithm to count pairs: [evidence span withheld — anchor status: not_found]
- [other] The chain-mesh method divides the catalogue into cubic cells and stores the indexes of all objects in each cell in vectors, so that to find all the objects close to a given one, the search is performed only on the relevant cells.: [evidence span withheld — anchor status: not_found]
- [readme] V3.0 Numerical Recipes fully substituted by GSL functions; classes to measure and model two-point statistics improved: "V3.0 Numerical Recipes fully substituted by GSL functions; classes to
     measure and model two-point statistics improved"
