---
name: jackknife-covariance-estimation
description: Use when you have a galaxy catalogue (real or simulated) and need covariance/error
  estimates for 2PCF measurements via jackknife, bootstrap, or lognormal mock resampling
  of survey sub-regions.
when_to_use_negative:
- You need full Python coverage of every CBL routine — only a subset of the CBL currently
  has a Python wrapper, so some steps must be driven from C++
- You require an archival, versioned release of the software — the article's download
  is a .tar archive from a personal institutional URL with no DOI or changelog
- The 2PCF example depends on local input files (cat.dat, random/, pairs/, output/
  directories) that are not described or hosted, so it cannot be reproduced from the
  paper alone
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: Cosmo Bologna Lib
  role: Provides the statistical-analysis classes for jackknife/bootstrap sub-sampling,
    catalogue handling, random-catalogue construction, and 2PCF measurement
  repo: https://github.com/federicomarulli/CosmoBolognaLib
- name: C++
  role: Primary language in which the CBL covariance and 2PCF machinery is written
    and invoked
- name: Python
  role: Alternative interface via SWIG wrappers built with 'make python'; only a subset
    of CBL functions is wrapped
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2016.01.005
    title: 'CosmoBolognaLib: C++ libraries for cosmological calculations'
  source_content_hash: sha256:f06f3a5e87887582b8df9c9dd3b468626a4bcde4b65ac580af5b3edb5a298b15
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
yes_rate: 0.9285714285714286
n_claims: 14
---

# jackknife-covariance-estimation

## Summary

Estimate covariance matrices and error bars for two-point correlation function (2PCF) measurements using internal sub-sampling — jackknife (delete-one-region) and bootstrap (regions drawn with replacement) — plus optional lognormal mocks, as implemented in the CosmoBolognaLib (CBL) statistical-analysis classes.

## When to use

Use when you have a galaxy catalogue (real or simulated) and need covariance/error estimates for 2PCF measurements via jackknife, bootstrap, or lognormal mock resampling of survey sub-regions.

## When NOT to use

- You need full Python coverage of every CBL routine — only a subset of the CBL currently has a Python wrapper, so some steps must be driven from C++
- You require an archival, versioned release of the software — the article's download is a .tar archive from a personal institutional URL with no DOI or changelog
- The 2PCF example depends on local input files (cat.dat, random/, pairs/, output/ directories) that are not described or hosted, so it cannot be reproduced from the paper alone

## Inputs

- Galaxy catalogue file (real or simulated, e.g. cat.dat handled via CBL Galaxy/Catalogue classes)
- Random catalogue or survey mask (MANGLE mask) defining sub-region geometry
- Input power spectrum (for lognormal mock catalogue generation)
- Compiled CBL libraries (libFUNC.so etc.) or Python wrappers

## Outputs

- Covariance matrix C_ij from jackknife resampling (Eq. 13 normalisation), e.g. covariance_matrix.dat
- Independent covariance/error estimate from bootstrap resampling
- 2PCF measurements recomputed under each sub-sampling realisation
- Optional: lognormal mock catalogues for external error comparison

## How to apply

Compile the CBL libraries from the github source with 'make lib*' (e.g. 'make libFUNC' to build libFUNC.so) and optionally 'make python' for the Python wrappers (a full SWIG wrapper is available from V1.1 onward). Load or generate a galaxy catalogue with CBL catalogue classes, then partition the survey volume into N sub-regions — either cubic sub-boxes or sub-regions defined by MANGLE survey masks. Recompute the 2PCF N times with the Two Point Correlation class, deleting one sub-region per iteration for jackknife, and assemble the covariance matrix C_ij using the Eq. 13 normalisation; repeat with bootstrap resampling (regions drawn with replacement) for an independent covariance estimate. Optionally build lognormal mock catalogues with an input power spectrum via the CBL random-catalogue machinery to provide external error estimates; note that the article states CBL provides 'a large set of methods ... to construct random catalogues, to estimate errors and to extract cosmological constraints from clustering analyses'.

## Related tools

- **Cosmo Bologna Lib** (Provides the statistical-analysis classes for jackknife/bootstrap sub-sampling, catalogue handling, random-catalogue construction, and 2PCF measurement) — https://github.com/federicomarulli/CosmoBolognaLib
- **C++** (Primary language in which the CBL covariance and 2PCF machinery is written and invoked)
- **Python** (Alternative interface via SWIG wrappers built with 'make python'; only a subset of CBL functions is wrapped)

## Examples

```
make libFUNC; make python
```

## Evaluation signals

- Output covariance_matrix.dat is square, symmetric, and positive semi-definite
- Diagonal errors of the covariance matrix are finite and non-negative
- Jackknife and bootstrap error bars agree within their expected statistical scatter on the synthetic catalogue
- Libraries compile cleanly with 'make lib*' (e.g. libFUNC.so is produced) and 'make python' builds the wrappers without error
- The 2PCF is recomputed exactly N times, once per deleted/drawn sub-region, before assembling C_ij

## Limitations

- Only a subset of the CBL has a Python wrapper; full coverage is deferred to a future version
- No fixed version/release identifier or archival DOI is given for the CBL download; the article points to a personal institutional URL and the repository has no changelog
- The 2PCF example relies on local input files (cat.dat and random/, pairs/, output/ directories) that are not described or hosted with the paper
- Example cosmological parameters use defaults unless set explicitly via a different constructor; details are deferred to external example codes on the CBL webpage

## Evidence

- [methods] The CBL estimate internal errors by sub-sampling the data catalogue and measuring the 2PCF for all but one region (jackknife), or for a random extraction of regions (bootstrap).: [evidence span withheld — anchor status: not_found]
- [intro] a large set of methods are provided to construct random catalogues, to estimate errors and to extract cosmological constraints from clustering analyses: [evidence span withheld — anchor status: not_found]
