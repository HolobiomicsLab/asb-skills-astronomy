---
name: two-point-correlation-function-estimation
description: Use when you need to measure the angle-averaged two-point correlation
  function of a galaxy or halo catalogue in configuration space from data and random
  catalogues over defined separation bins (e.g. r = 1–50 h⁻¹Mpc).
when_to_use_negative:
- When the goal is the three-point correlation function — that requires the Szapudi-Szalay
  estimator with DDD/DDR/DRR/RRR triplet counts, not pair counting
- When only a subset of the analysis is needed in Python — the Python wrapper covers
  only a subset of CBL functions, so C++ may be required
- When the task is modelling rather than measuring ξ(r) — the redshift-space 2PCF
  model requires the Cosmology class and Fourier anti-transform of the de-wiggled
  P(k), not the measurement pipeline
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: CosmoBolognaLib
  role: C++ libraries providing the TwoPointCorrelation classes, Catalogue/RandomCatalogue
    handling, and the parallel chain-mesh pair-counting algorithm
  repo: https://github.com/federicomarulli/CosmoBolognaLib
- name: GSL
  role: External numerical library backing CBL's two-point statistics computations
    (fully substituted for Numerical Recipes from V3.0)
- name: Python
  role: Optional high-level scripting access to CBL through SWIG wrappers compiled
    with 'make python'
- name: C++
  role: Primary language in which the TwoPointCorrelation example programs are written
    and compiled
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_007
  source_papers:
  - doi: 10.1016/j.ascom.2016.01.005
    title: 'CosmoBolognaLib: C++ libraries for cosmological calculations'
  source_content_hash: sha256:fcd172e60b7c12d1e32c1a93a121f0f72b75e624b92793679d3309aaf5fb6585
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# two-point-correlation-function-estimation

## Summary

Measure the angle-averaged galaxy two-point correlation function ξ(r) with the CosmoBolognaLib (CBL) TwoPointCorrelation classes, counting DD, DR and RR pairs using the parallel chain-mesh algorithm and combining them with the Landy-Szalay estimator.

## When to use

Use when you need to measure the angle-averaged two-point correlation function of a galaxy or halo catalogue in configuration space from data and random catalogues over defined separation bins (e.g. r = 1–50 h⁻¹Mpc).

## When NOT to use

- When the goal is the three-point correlation function — that requires the Szapudi-Szalay estimator with DDD/DDR/DRR/RRR triplet counts, not pair counting
- When only a subset of the analysis is needed in Python — the Python wrapper covers only a subset of CBL functions, so C++ may be required
- When the task is modelling rather than measuring ξ(r) — the redshift-space 2PCF model requires the Cosmology class and Fourier anti-transform of the de-wiggled P(k), not the measurement pipeline

## Inputs

- Galaxy catalogue file (e.g. cat.dat) readable into a CBL Catalogue object
- Associated random catalogue (constructed via CBL random-catalogue algorithms)
- Separation-bin configuration (linearly spaced bins, e.g. r = 1–50 h⁻¹Mpc)
- Compiled CosmoBolognaLib with GSL, FFTW and OpenMP installed

## Outputs

- DD, DR and RR pair-count files written under the pairs/ directory
- Measured ξ(r) table written as output/xi.dat with values at each bin centre

## How to apply

Load the compiled CosmoBolognaLib (C++ libraries relying on GSL, with optional SWIG Python wrappers) and include the TwoPointCorrelation classes in the example program. Read the galaxy catalogue file (cat.dat) into a Catalogue object and construct the associated random catalogue using CBL's random-catalogue algorithms. Configure an angle-averaged TwoPointCorrelation1D object with linearly spaced separation bins spanning r = 1–50 h⁻¹Mpc, then count data–data (DD), data–random (DR) and random–random (RR) pairs with CBL's parallel chain-mesh pair-counting algorithm, writing pair files under the pairs/ directory. Combine the normalised pair counts with the Landy-Szalay estimator to obtain ξ(r) at each bin centre, and write the measured ξ(r) table into output/xi.dat.

## Related tools

- **CosmoBolognaLib** (C++ libraries providing the TwoPointCorrelation classes, Catalogue/RandomCatalogue handling, and the parallel chain-mesh pair-counting algorithm) — https://github.com/federicomarulli/CosmoBolognaLib
- **GSL** (External numerical library backing CBL's two-point statistics computations (fully substituted for Numerical Recipes from V3.0))
- **Python** (Optional high-level scripting access to CBL through SWIG wrappers compiled with 'make python')
- **C++** (Primary language in which the TwoPointCorrelation example programs are written and compiled)

## Examples

```
enter the Cosmo_Bologna_Lib/ folder and type: make   # then compile the TwoPointCorrelation1D example with linearly spaced bins over r = 1-50 h^-1Mpc and run it on cat.dat
```

## Evaluation signals

- output/xi.dat exists and contains ξ(r) values for every bin in the 1–50 h⁻¹Mpc range
- The measured ξ(r) curve decreases smoothly at large separations, as expected for a clustered galaxy sample
- DD, DR and RR pair-count files are present under the pairs/ directory for all configured bins
- Pair counts are consistent between the data and random catalogues' normalisations (Landy-Szalay combination yields finite, positive ξ(r) at small separations)

## Limitations

- The 2PCF example depends on local input files (cat.dat, random/, pairs/, output/ directories) that are not described or hosted with the paper
- Only a subset of the CBL has a Python wrapper; full coverage is deferred to a future version, so C++ may be needed
- No fixed version/release identifier is given for the CBL download in the paper; only a .tar archive at a personal institutional URL (the GitHub repo tracks versions V1.0 through V5.2)
- Version-sensitive behaviour: V3.0 substituted GSL for Numerical Recipes, and V4.1 onward significantly improved the two-point and three-point correlation-function modelling classes

## Evidence

- [intro] the CBL provide highly optimised algorithms to measure two-point (2PCF) and three-point correlation functions (3PCF), exploiting a specifically designed parallel chain-mesh algorithm to count pairs: [evidence span withheld — anchor status: not_found]
- [supplementary] The implemented Landy-Szalay estimator combines the normalised data-data (DD), random-random (RR) and data-random (DR) pair counts computed for separation bins of the 2PCF.: [evidence span withheld — anchor status: not_found]
