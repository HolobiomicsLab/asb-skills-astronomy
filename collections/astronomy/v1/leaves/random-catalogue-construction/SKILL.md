---
name: random-catalogue-construction
description: 'Use when measuring a two-point correlation function from a galaxy catalogue:
  a matching random catalogue must first be built using CBL''s random-catalogue algorithms
  before DR and RR pair counting.'
when_to_use_negative:
- The analysis target is one-point or three-point statistics, or cosmological parameter
  extraction, rather than two-point clustering measurements that require random catalogues
- The galaxy catalogue already ships with an official random catalogue that matches
  the survey geometry, making a new random-catalogue construction redundant
- 'Only a subset of functionality is needed in Python: CBL''s Python wrapper covers
  only a subset of functions, so full random-catalogue algorithms may require C++'
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Cosmo Bologna Lib
  role: Provides the classes and random-catalogue construction algorithms and the
    parallel chain-mesh pair-counting used to compute DR/RR pairs for the 2PCF
  repo: https://github.com/federicomarulli/CosmoBolognaLib
- name: C++
  role: Language in which the CBL libraries are fully written; random-catalogue construction
    is invoked from C++ codes (Python wrapper only covers a subset)
- name: GSL
  role: External library required by CBL's two-point statistics classes; since V3.0
    Numerical Recipes was fully substituted by GSL functions
- name: Python
  role: Optional high-level access to CBL classes via SWIG wrappers, compiled with
    'make python'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2016.01.005
    title: 'CosmoBolognaLib: C++ libraries for cosmological calculations'
  source_content_hash: sha256:02dca095f584e6bdcb725885a96111323b792a55754fdfe188f3c7ecb2d3129a
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# random-catalogue-construction

## Summary

Generate the random (homogeneous mock) catalogue that accompanies a galaxy catalogue, required to compute DD, DR and RR pair counts for two-point correlation function estimators such as Landy-Szalay with the CosmoBolognaLib (CBL).

## When to use

Use when measuring a two-point correlation function from a galaxy catalogue: a matching random catalogue must first be built using CBL's random-catalogue algorithms before DR and RR pair counting.

## When NOT to use

- The analysis target is one-point or three-point statistics, or cosmological parameter extraction, rather than two-point clustering measurements that require random catalogues
- The galaxy catalogue already ships with an official random catalogue that matches the survey geometry, making a new random-catalogue construction redundant
- Only a subset of functionality is needed in Python: CBL's Python wrapper covers only a subset of functions, so full random-catalogue algorithms may require C++

## Inputs

- Galaxy catalogue file (e.g. cat.dat) loaded into a CBL Catalogue object
- Survey geometry and sky coverage parameters defining the random catalogue volume
- Target random catalogue size (number of random points relative to data points)
- Separation-bin configuration for pair counting (e.g. linearly spaced bins over 1–50 h⁻¹Mpc)

## Outputs

- Random catalogue object compatible with CBL TwoPointCorrelation classes
- DR and RR pair-count files written under the pairs/ directory
- Normalised pair counts feeding the Landy-Szalay estimator for ξ(r)

## How to apply

Install the CosmoBolognaLib (C++ libraries that rely on GSL, FFTW and OpenMP; from version 3.0 Numerical Recipes was fully substituted by GSL functions) and compile them with 'make' in the CosmoBolognaLib/ folder, adding 'make python' if wrappers are needed. Read the galaxy catalogue (e.g. cat.dat) into a CBL Catalogue object, then construct the associated random catalogue using the CBL random-catalogue algorithms, sized appropriately for the survey geometry and number density so the DR and RR pair counts are statistically well-sampled. Count data-random (DR) and random-random (RR) pairs with CBL's parallel chain-mesh pair-counting algorithm over the same separation bins (e.g. linearly spaced bins spanning r = 1–50 h⁻¹Mpc) used for the data-data (DD) counts, writing pair files under the pairs/ directory. The rationale is that the Landy-Szalay estimator combines normalised DD, RR and DR pair counts, so the random catalogue's geometry and density directly determine the fidelity of ξ(r) at every bin centre.

## Related tools

- **Cosmo Bologna Lib** (Provides the classes and random-catalogue construction algorithms and the parallel chain-mesh pair-counting used to compute DR/RR pairs for the 2PCF) — https://github.com/federicomarulli/CosmoBolognaLib
- **C++** (Language in which the CBL libraries are fully written; random-catalogue construction is invoked from C++ codes (Python wrapper only covers a subset))
- **GSL** (External library required by CBL's two-point statistics classes; since V3.0 Numerical Recipes was fully substituted by GSL functions)
- **Python** (Optional high-level access to CBL classes via SWIG wrappers, compiled with 'make python')

## Examples

```
cd CosmoBolognaLib/ && make && make python
```

## Evaluation signals

- Random catalogue geometry matches the parent galaxy catalogue (same survey mask and area, e.g. the HOD catalogue volume of 3.677700e+07 [Mpc/h]^3 for a 1/8 sphere)
- DR and RR pair-count files are produced under the pairs/ directory for every separation bin in the configured range (1–50 h⁻¹Mpc)
- The resulting ξ(r) curve from the Landy-Szalay estimator decreases smoothly at large separations, as expected for a clustered galaxy sample
- RR pair counts scale as the square of the random catalogue size and remain well above Poisson noise relative to DR counts

## Limitations

- Only a subset of the CBL has currently a Python wrapper; full coverage of random-catalogue and clustering functions is deferred to future versions
- The 2PCF example depends on local input files (cat.dat and the random/, pairs/, output/ directories) that are not described or hosted by the article
- No fixed version/release identifier or archival DOI is given for the CBL download; only a .tar archive at a personal institutional URL
- The repository contains no changelog

## Evidence

- [other] CBL random-catalogue algorithms: [evidence span withheld — anchor status: not_found]
- [intro] random catalogue methods in CBL: [evidence span withheld — anchor status: not_found]
- [intro] parallel chain-mesh pair counting: [evidence span withheld — anchor status: not_found]
