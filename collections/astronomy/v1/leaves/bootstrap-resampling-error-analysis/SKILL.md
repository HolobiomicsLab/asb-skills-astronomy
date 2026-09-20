---
name: bootstrap-resampling-error-analysis
description: Use when quantifying uncertainty on 2PCF (or other clustering) measurements
  from a galaxy catalogue without external mocks, via jackknife/bootstrap sub-region
  resampling in CosmoBolognaLib.
when_to_use_negative:
- When the analysis goal is modelling or cosmological parameter extraction from the
  2PCF rather than error estimation on the measurement itself
- When external mock catalogues (not internal sub-sampling) are the required error
  method and no lognormal mock machinery is needed
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: Cosmo Bologna Lib
  role: Provides the catalogue classes, sub-region partitioning, jackknife/bootstrap
    resampling, and covariance-assembly statistical-analysis classes used to estimate
    2PCF errors
  repo: github.com/federicomarulli/CosmoBolognaLib
- name: C++
  role: Primary language in which the CBL libraries are written and compiled (e.g.
    libFUNC.so via 'make libFUNC')
- name: Python
  role: High-level wrapper language for driving CBL resampling and covariance estimation
    from scripts
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2016.01.005
    title: 'CosmoBolognaLib: C++ libraries for cosmological calculations'
  source_content_hash: sha256:417b8b9fbd2adfb813c8f963a88710c31be9a7a4b9e74859f1fed2197f554803
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - C++
  - Cosmo Bologna Lib
  - Python
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# bootstrap-resampling-error-analysis

## Summary

Estimate internal errors on two-point correlation function (2PCF) measurements by sub-sampling the survey volume into N sub-regions and re-measuring clustering under jackknife (delete-one) and bootstrap (draw-with-replacement) resampling, using the CosmoBolognaLib statistical-analysis classes to assemble covariance matrices.

## When to use

Use when quantifying uncertainty on 2PCF (or other clustering) measurements from a galaxy catalogue without external mocks, via jackknife/bootstrap sub-region resampling in CosmoBolognaLib.

## When NOT to use

- When the analysis goal is modelling or cosmological parameter extraction from the 2PCF rather than error estimation on the measurement itself
- When external mock catalogues (not internal sub-sampling) are the required error method and no lognormal mock machinery is needed

## Inputs

- Galaxy catalogue (real or synthetic) loadable via CBL catalogue classes (e.g. cat.dat with random/ catalogue files)
- Sub-region definition: cubic sub-boxes or MANGLE survey masks partitioning the survey volume
- Input power spectrum for optional lognormal mock catalogue generation

## Outputs

- covariance_matrix.dat: square, symmetric, positive semi-definite covariance matrix C_ij of 2PCF errors
- Diagonal error bars on the 2PCF from jackknife and bootstrap resampling

## How to apply

Compile the needed CosmoBolognaLib modules with 'make lib*' (e.g. 'make libFUNC' to build libFUNC.so) and optionally 'make python' to drive the machinery from Python. Load a synthetic or real galaxy catalogue through the CBL catalogue classes, then partition the survey volume into N sub-regions, either as cubic sub-boxes or using MANGLE survey masks to define the sub-region geometry. Recompute the clustering measurement N times under jackknife resampling (deleting one sub-region at a time) and assemble the covariance matrix C_ij using the Eq. 13 normalisation; then repeat the resampling with bootstrap (sub-regions drawn with replacement) to obtain an independent covariance estimate from the same CBL statistical-analysis classes. Optionally generate lognormal mock catalogues with an input power spectrum via the CBL random-catalogue machinery to provide external error estimates on the covariance. Validate that the output covariance_matrix.dat is square, symmetric, and positive semi-definite, and that jackknife and bootstrap error bars agree within their expected statistical scatter.

## Related tools

- **Cosmo Bologna Lib** (Provides the catalogue classes, sub-region partitioning, jackknife/bootstrap resampling, and covariance-assembly statistical-analysis classes used to estimate 2PCF errors) — github.com/federicomarulli/CosmoBolognaLib
- **C++** (Primary language in which the CBL libraries are written and compiled (e.g. libFUNC.so via 'make libFUNC'))
- **Python** (High-level wrapper language for driving CBL resampling and covariance estimation from scripts)

## Examples

```
make libFUNC; make python
```

## Evaluation signals

- Output covariance_matrix.dat is square and symmetric
- Covariance matrix is positive semi-definite
- Diagonal errors of the covariance matrix are finite and non-negative
- Jackknife and bootstrap error bars agree within their expected statistical scatter on the synthetic catalogue
- Libraries compile successfully (e.g. libFUNC.so built with 'make libFUNC') before running the resampling

## Limitations

- Only a subset of the CBL has a Python wrapper; full coverage deferred to a future version, so some steps may require C++
- No fixed version/release identifier given for the CBL download in the article; only a .tar archive at a personal institutional URL
- The 2PCF example depends on local input files (cat.dat, random/, pairs/, output/ directories) not described or hosted in the paper

## Evidence

- [methods] The CBL estimate internal errors by sub-sampling the data catalogue and measuring the 2PCF for all but one region (jackknife), or for a random extraction of regions (bootstrap).: [evidence span withheld — anchor status: not_found]
- [intro] a large set of methods are provided to construct random catalogues, to estimate errors and to extract cosmological constraints from clustering analyses: [evidence span withheld — anchor status: not_found]
- [abstract] help in handling astronomical catalogues, both real and simulated, measuring one-point, two-point and three-point statistics in configuration space: "one of the primary focuses of this software is to help in handling astronomical catalogues, both real and simulated, measuring one-point, two-point and three-point statistics in configuration space,"
- [discussion] make lib* → compile the * library (e.g. make libFUNC will compile the library libFUNC.so): "make lib* → compile the * library (e.g. make libFUNC will compile the library libFUNC.so)"
- [readme] The example codes in CosmoBolognaLib/Examples/ show how to use the CosmoBolognaLib in either C++ or Python codes.: "The example codes in CosmoBolognaLib/Examples/ show how to use the CosmoBolognaLib in either C++ or Python codes."
