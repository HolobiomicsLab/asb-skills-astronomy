---
name: survey-volume-subregion-partitioning
description: Use when estimating error bars or a covariance matrix C_ij on a two-point
  correlation function measurement, and the survey volume can be subdivided into N
  spatially distinct sub-regions for delete-one or with-replacement resampling.
when_to_use_negative:
- The analysis goal is a single 2PCF measurement without uncertainties — sub-sampling
  covariance machinery adds cost with no benefit
- The survey volume cannot be meaningfully partitioned (e.g. sparse or irregular catalogues
  where delete-one regions produce unstable estimates)
- The task requires full CBL Python coverage — only a subset of CBL functions has
  a Python wrapper, so some steps may need C++
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: Cosmo Bologna Lib
  role: Provides the catalogue classes, statistical-analysis classes, and random-catalogue
    machinery used to partition the volume and run jackknife/bootstrap 2PCF covariance
    estimation
  repo: https://github.com/federicomarulli/CosmoBolognaLib
- name: C++
  role: Primary language in which the CBL libraries are written and compiled (e.g.
    'make libFUNC' builds libFUNC.so)
- name: Python
  role: Alternative interface for driving the CBL covariance/2PCF machinery via SWIG-built
    wrappers ('make python')
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2016.01.005
    title: 'CosmoBolognaLib: C++ libraries for cosmological calculations'
  source_content_hash: sha256:f0af06f1d339c1ea59df46d731d404ff0d2b97739fbee65e06f57cadb2216ce2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/federicomarulli/CosmoBolognaLib
  tools:
  - C++
  - Cosmo Bologna Lib
  - Python
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# survey-volume-subregion-partitioning

## Summary

Partition a galaxy survey volume into N sub-regions (cubic sub-boxes or MANGLE survey mask geometry) so that internal sub-sampling techniques — jackknife (delete-one-region) and bootstrap (regions drawn with replacement) — can be applied by CosmoBolognaLib classes to estimate 2PCF covariance matrices.

## When to use

Use when estimating error bars or a covariance matrix C_ij on a two-point correlation function measurement, and the survey volume can be subdivided into N spatially distinct sub-regions for delete-one or with-replacement resampling.

## When NOT to use

- The analysis goal is a single 2PCF measurement without uncertainties — sub-sampling covariance machinery adds cost with no benefit
- The survey volume cannot be meaningfully partitioned (e.g. sparse or irregular catalogues where delete-one regions produce unstable estimates)
- The task requires full CBL Python coverage — only a subset of CBL functions has a Python wrapper, so some steps may need C++

## Inputs

- Galaxy catalogue file (e.g. cat.dat) loadable by CBL catalogue classes (real or simulated extragalactic sources)
- Random catalogue for pair counting (random/ directory inputs used by the 2PCF example)
- MANGLE survey masks defining sub-region geometry (or cubic sub-box subdivision parameters)
- Input power spectrum for optional lognormal mock catalogue generation

## Outputs

- covariance_matrix.dat: square, symmetric, positive semi-definite covariance matrix C_ij for the 2PCF
- Jackknife error bars (delete-one-region 2PCF estimates with Eq. 13 normalisation)
- Independent bootstrap error bars (regions drawn with replacement)
- Optional lognormal mock catalogue-based external error estimates

## How to apply

First build the needed CBL libraries from the github source (e.g. 'make libFUNC' to build libFUNC.so, optionally 'make python' for the SWIG-based Python wrappers). Load or generate a synthetic or real galaxy catalogue with CBL catalogue classes in the common CBL environment for handling extragalactic sources. Partition the survey volume into N sub-regions, choosing cubic sub-boxes or MANGLE survey masks as the sub-region geometry. Recompute the clustering measurement N times under jackknife resampling (deleting one sub-region at a time) and assemble the covariance matrix C_ij using the Eq. 13 normalisation; then repeat with bootstrap resampling (sub-regions drawn with replacement) for an independent covariance estimate. Optionally generate lognormal mock catalogues with an input power spectrum via the CBL random-catalogue machinery to cross-check the internal error estimates.

## Related tools

- **Cosmo Bologna Lib** (Provides the catalogue classes, statistical-analysis classes, and random-catalogue machinery used to partition the volume and run jackknife/bootstrap 2PCF covariance estimation) — https://github.com/federicomarulli/CosmoBolognaLib
- **C++** (Primary language in which the CBL libraries are written and compiled (e.g. 'make libFUNC' builds libFUNC.so))
- **Python** (Alternative interface for driving the CBL covariance/2PCF machinery via SWIG-built wrappers ('make python'))

## Examples

```
make libFUNC && make python
```

## Evaluation signals

- Output covariance_matrix.dat is square and symmetric
- Covariance matrix is positive semi-definite
- Diagonal error entries are finite and non-negative
- Jackknife and bootstrap error bars agree within their expected statistical scatter on the synthetic catalogue

## Limitations

- Only a subset of the CBL has a Python wrapper; full coverage is deferred to a future version, so some steps may require C++ code
- No fixed version/release identifier is given for the CBL download — only a .tar archive at a personal institutional URL with no DOI/Zenodo, and the repository contains no changelog
- The 2PCF example depends on local input files (cat.dat, random/, pairs/, output/ directories) that are not described or hosted, so reproducing the exact setup requires constructing these inputs yourself

## Evidence

- [other] The CBL estimate internal errors by sub-sampling the data catalogue and measuring the 2PCF for all but one region (jackknife), or for a random extraction of regions (bootstrap).: [evidence span withheld — anchor status: not_found]
- [intro] a large set of methods are provided to construct random catalogues, to estimate errors and to extract cosmological constraints from clustering analyses: [evidence span withheld — anchor status: not_found]
