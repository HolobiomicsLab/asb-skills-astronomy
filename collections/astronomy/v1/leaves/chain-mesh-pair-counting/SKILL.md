---
name: chain-mesh-pair-counting
description: Use when measuring angle-averaged two-point or three-point correlation
  functions from galaxy/simulated halo catalogues and their random catalogues in configuration
  space, and you need optimised parallel pair/triplet counting.
when_to_use_negative:
- The analysis needs Fourier-space (Pk) measurements rather than configuration-space
  pair counts
- The catalogue is small enough that naive O(N²) pair counting is trivially fast,
  and integrating the full CBL build (GSL, FFTW, OpenMP, Numerical Recipes) is not
  worthwhile
- 'You need a function that is only in C++: the Python wrapper covers only a subset
  of CBL, so wrapper-only pipelines may not reach the needed class'
edam_operation: http://edamontology.org/operation_3465
tools:
- name: CosmoBolognaLib
  role: Provides the TwoPointCorrelation/ThreePointCorrelation classes and the parallel
    chain-mesh pair/triplet counting algorithm
  repo: https://github.com/federicomarulli/CosmoBolognaLib
- name: C++
  role: Primary language in which CBL chain-mesh counting is compiled and invoked
- name: GSL
  role: External numerical library the two-point statistics classes rely on (fully
    substituted for Numerical Recipes from V3.0)
- name: Python
  role: Optional SWIG-wrapped interface for calling the counting and estimator classes
    from Python codes
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2016.01.005
    title: 'CosmoBolognaLib: C++ libraries for cosmological calculations'
  source_content_hash: sha256:cfa0a3c130b096364e04d8c69853b650becee2d15403f4a437bd76ce6cf10c0c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/federicomarulli/CosmoBolognaLib
  tools:
  - C++
  - Cosmo Bologna Lib
  - CosmoBolognaLib
  - GSL
  - Python
  edam_operation: http://edamontology.org/operation_3465
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# chain-mesh-pair-counting

## Summary

Use CosmoBolognaLib's parallel chain-mesh algorithm to count data-data, data-random and random-random pairs (and triplets) in separation bins, as the numerics engine behind Landy-Szalay (2PCF) and Szapudi-Szalay (3PCF) estimators in C++ or Python.

## When to use

Use when measuring angle-averaged two-point or three-point correlation functions from galaxy/simulated halo catalogues and their random catalogues in configuration space, and you need optimised parallel pair/triplet counting.

## When NOT to use

- The analysis needs Fourier-space (Pk) measurements rather than configuration-space pair counts
- The catalogue is small enough that naive O(N²) pair counting is trivially fast, and integrating the full CBL build (GSL, FFTW, OpenMP, Numerical Recipes) is not worthwhile
- You need a function that is only in C++: the Python wrapper covers only a subset of CBL, so wrapper-only pipelines may not reach the needed class

## Inputs

- Galaxy or simulated halo catalogue file (e.g. cat.dat) loaded into a CBL Catalogue object
- Associated random catalogue constructed with CBL random-catalogue algorithms
- Separation-bin configuration (linear bins, e.g. r = 1–50 h⁻¹Mpc) for 2PCF
- Triangle-side configuration (fixed r12, r23, r31; varying angle θ) for 3PCF

## Outputs

- DD, DR, RR pair-count files under the pairs/ directory
- Measured correlation function table output/xi.dat with ξ(r) per bin centre
- 3PCF output table with ζ and reduced Q columns as a function of θ

## How to apply

Load the compiled CosmoBolognaLib (C++ libraries backed by GSL, with optional SWIG Python wrappers) and read the galaxy catalogue (e.g. cat.dat) into a Catalogue object, constructing an associated random catalogue with CBL's random-catalogue algorithms. Configure an angle-averaged TwoPointCorrelation1D object with linearly spaced separation bins (e.g. r = 1–50 h⁻¹Mpc) and run the parallel chain-mesh pair-counting algorithm to count DD, DR and RR pairs, writing pair files under pairs/. Combine the normalised counts with the Landy-Szalay estimator (Eq. 4) to obtain ξ(r) at each bin centre; for 3PCF, count DDD/DDR/DRR/RRR triplets at fixed triangle sides as a function of opening angle θ and combine them via the Szapudi-Szalay estimator (Eq. 11), optionally computing the reduced 3PCF Q = ζ/(ξ12ξ23 + ξ23ξ31 + ξ31ξ12). Rationale: CBL provides specifically designed, highly optimised chain-mesh algorithms so that pair and triplet counting is the computational bottleneck handled by the library, not hand-written loops.

## Related tools

- **CosmoBolognaLib** (Provides the TwoPointCorrelation/ThreePointCorrelation classes and the parallel chain-mesh pair/triplet counting algorithm) — https://github.com/federicomarulli/CosmoBolognaLib
- **C++** (Primary language in which CBL chain-mesh counting is compiled and invoked)
- **GSL** (External numerical library the two-point statistics classes rely on (fully substituted for Numerical Recipes from V3.0))
- **Python** (Optional SWIG-wrapped interface for calling the counting and estimator classes from Python codes)

## Examples

```
Catalogue gal{cbl::Var__X_, cbl::Var__Y_, cbl::Var__Z_, cbl::Var__Mass_, catFile}; TwoPointCorrelation1D twopt{gal, random, binType, rMin, rMax, nbins, 0, 0, 0}; twopt.count_pairs(cbl::ChainMesh_); twopt.get_xi(); twopt.write(outputDir, "xi.dat");
```

## Evaluation signals

- output/xi.dat exists and contains ξ(r) values for every bin in the 1–50 h⁻¹Mpc range
- The measured ξ(r) curve decreases smoothly at large separations, as expected for a clustered galaxy sample
- The 3PCF output table contains ζ and Q columns in the estimator-derived format with finite values
- The reduced 3PCF Q is of order unity across the θ bins
- DD, DR and RR pair files are produced under pairs/ for every configured separation bin

## Limitations

- Only a subset of the CBL has a Python wrapper; full coverage is deferred to a future version
- The 2PCF example depends on local input files (cat.dat, random/, pairs/, output/ directories) not described or hosted with the paper
- No fixed version/release identifier is given for the download; only a .tar archive at a personal institutional URL (no DOI/Zenodo), and the repository contains no changelog

## Evidence

- [intro] the CBL provide highly optimised algorithms to measure two-point (2PCF) and three-point correlation functions (3PCF), exploiting a specifically designed parallel chain-mesh algorithm to count pairs: [evidence span withheld — anchor status: not_found]
- [other] The implemented Landy-Szalay estimator combines the normalised data-data (DD), random-random (RR) and data-random (DR) pair counts computed for separation bins of the 2PCF.: [evidence span withheld — anchor status: not_found]
- [readme] Numerical Recipes fully substituted by GSL functions; classes to measure and model two-point statistics improved: "V3.0 Numerical Recipes fully substituted by GSL functions; classes to
     measure and model two-point statistics improved"
- [intro] a large set of methods are provided to construct random catalogues, to estimate errors and to extract cosmological constraints from clustering analyses: [evidence span withheld — anchor status: not_found]
