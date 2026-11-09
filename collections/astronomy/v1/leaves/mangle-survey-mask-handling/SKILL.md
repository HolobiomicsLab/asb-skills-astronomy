---
name: mangle-survey-mask-handling
description: Use when a galaxy survey footprint must be split into N sub-regions for
  jackknife or bootstrap covariance estimation and the geometry follows the survey
  mask (MANGLE) rather than a periodic-box cubic decomposition.
when_to_use_negative:
- The survey volume is a simulation box with no angular mask, where simple cubic sub-box
  partitioning suffices
- Errors are to be estimated externally from lognormal mock catalogues via the CBL
  random-catalogue machinery rather than from internal sub-sampling
- The target statistic is a one-point or three-point measurement rather than the two-point
  correlation function covariance
tools:
- name: Cosmo Bologna Lib
  role: Provides the catalogue-handling and statistical-analysis classes that partition
    the survey volume using MANGLE masks and recompute the 2PCF under jackknife/bootstrap
    resampling
  repo: github.com/federicomarulli/CosmoBolognaLib
- name: C++
  role: Native language of the CBL libraries used for the mask-based partitioning
    and pair-counting chain-mesh algorithms
- name: Python
  role: Optional driver of the CBL machinery via the compiled Python wrappers ('make
    python')
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2016.01.005
    title: 'CosmoBolognaLib: C++ libraries for cosmological calculations'
  source_content_hash: sha256:66ab5b6aef6c0a12c23be6a65fe172129b9e93bf70d81170a4d2e48f8b933ad4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - C++
  - Cosmo Bologna Lib
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# mangle-survey-mask-handling

## Summary

Defines the sub-region geometry for internal error estimation (jackknife/bootstrap covariance matrices on 2PCF measurements) by partitioning the survey volume using MANGLE survey masks, as an alternative to simple cubic sub-boxes, within the CosmoBolognaLib statistical-analysis environment.

## When to use

Use when a galaxy survey footprint must be split into N sub-regions for jackknife or bootstrap covariance estimation and the geometry follows the survey mask (MANGLE) rather than a periodic-box cubic decomposition.

## When NOT to use

- The survey volume is a simulation box with no angular mask, where simple cubic sub-box partitioning suffices
- Errors are to be estimated externally from lognormal mock catalogues via the CBL random-catalogue machinery rather than from internal sub-sampling
- The target statistic is a one-point or three-point measurement rather than the two-point correlation function covariance

## Inputs

- Galaxy catalogue (real or simulated) loadable by CBL catalogue classes
- MANGLE survey mask defining the survey footprint and sub-region geometry
- Compiled CosmoBolognaLib shared libraries (e.g. libFUNC.so) built with GSL, FFTW and OpenMP available

## Outputs

- N mask-defined sub-regions of the survey volume
- N resampled 2PCF measurements (one per sub-region deletion/draw)
- Covariance matrix written to covariance_matrix.dat (square, symmetric, positive semi-definite, with diagonal jackknife/bootstrap errors)

## How to apply

Compile the relevant CosmoBolognaLib libraries (e.g. 'make libFUNC', optionally 'make python' for the wrappers) after installing the required external dependencies (GSL, FFTW, OpenMP). Load or generate a galaxy catalogue with CBL catalogue classes, which provide a common environment for handling real and simulated extragalactic source catalogues. Instead of cutting the volume into cubic sub-boxes, use the MANGLE survey mask to define the sub-region geometry so each region follows the survey footprint; then recompute the 2PCF N times with jackknife (delete one region at a time) and assemble the covariance matrix C_ij with the Eq. 13 normalisation. Bootstrap resampling (regions drawn with replacement) over the same mask-defined regions gives an independent covariance estimate. The rationale is that mask-based partitioning respects survey boundaries and selection geometry, which cubic sub-boxes ignore.

## Related tools

- **Cosmo Bologna Lib** (Provides the catalogue-handling and statistical-analysis classes that partition the survey volume using MANGLE masks and recompute the 2PCF under jackknife/bootstrap resampling) — github.com/federicomarulli/CosmoBolognaLib
- **C++** (Native language of the CBL libraries used for the mask-based partitioning and pair-counting chain-mesh algorithms)
- **Python** (Optional driver of the CBL machinery via the compiled Python wrappers ('make python'))

## Examples

```
make libFUNC && make python
```

## Evaluation signals

- The MANGLE-mask-defined sub-regions jointly cover the full survey footprint without overlap, and N distinct resamples are produced
- covariance_matrix.dat is square and symmetric and passes a positive semi-definiteness check
- Diagonal covariance errors are finite and non-negative
- Jackknife and bootstrap error bars agree within their expected statistical scatter on the synthetic catalogue
- Mask-based sub-region geometry reproduces the survey footprint, unlike naive cubic sub-boxes which would ignore selection geometry

## Limitations

- Only a subset of the CBL functions has a Python wrapper; full coverage is deferred to a future version of the libraries
- The example workflow depends on local input files (cat.dat, random/, pairs/, output/ directories) not described or hosted with the paper
- No fixed version/release identifier or archival DOI is given for the CBL download; only a GitHub repository and a .tar archive are referenced

## Evidence

- [methods] Partition the survey volume into N sub-regions, either as cubic sub-boxes or using MANGLE survey masks to define the sub-region geometry.: [evidence span withheld — anchor status: not_found]
- [intro] a large set of methods are provided to construct random catalogues, to estimate errors and to extract cosmological constraints from clustering analyses: [evidence span withheld — anchor status: not_found]
