---
name: lognormal-mock-catalogue-generation
description: Use when you need synthetic mock catalogues with realistic clustering
  (driven by an input power spectrum) to validate error estimates, e.g. jackknife/bootstrap
  covariance matrices for 2PCF measurements.
when_to_use_negative:
- You need full-survey data covariance from actual survey masks only and have no use
  for synthetic power-spectrum-driven mocks
- The analysis requires N-body or halo-occupation mocks beyond the lognormal approximation
  (CBL focuses on handling real and simulated catalogues, but the workflow here is
  specifically lognormal mocks)
- You are working outside the CBL environment and cannot compile its C++ libraries
  or use the (incomplete) Python wrapper subset
edam_operation: http://edamontology.org/operation_2426
tools:
- name: Cosmo Bologna Lib
  role: 'Core library: random-catalogue machinery to generate lognormal mocks from
    an input power spectrum, and Two Point Correlation classes to measure clustering
    on them'
  repo: https://github.com/federicomarulli/CosmoBolognaLib
- name: C++
  role: Language in which CBL is fully written; used to drive mock generation directly
- name: Python
  role: Alternative driver language via the CBL Python wrappers built with 'make python'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2016.01.005
    title: 'CosmoBolognaLib: C++ libraries for cosmological calculations'
  source_content_hash: sha256:02df98a18fb4a2e7a3b3a66cbf2e28e15e06d0f67dbf6519e47a15617dba72cf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/federicomarulli/CosmoBolognaLib
  tools:
  - C++
  - Cosmo Bologna Lib
  - Python
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# lognormal-mock-catalogue-generation

## Summary

Generate synthetic lognormal galaxy mock catalogues from an input power spectrum using the CosmoBolognaLib (CBL) random-catalogue machinery, providing external error estimates on clustering measurements such as 2PCF covariance matrices.

## When to use

Use when you need synthetic mock catalogues with realistic clustering (driven by an input power spectrum) to validate error estimates, e.g. jackknife/bootstrap covariance matrices for 2PCF measurements.

## When NOT to use

- You need full-survey data covariance from actual survey masks only and have no use for synthetic power-spectrum-driven mocks
- The analysis requires N-body or halo-occupation mocks beyond the lognormal approximation (CBL focuses on handling real and simulated catalogues, but the workflow here is specifically lognormal mocks)
- You are working outside the CBL environment and cannot compile its C++ libraries or use the (incomplete) Python wrapper subset

## Inputs

- Input power spectrum (matter or galaxy) for the lognormal realisation
- Cosmological parameter set (e.g. Omega_M, Omega_DE, sigma8, h) configured in a CBL Cosmology object
- Survey geometry / mask specification (e.g. cubic sub-boxes or MANGLE survey masks) defining the catalogue volume
- Galaxy catalogue file such as cat.dat used in CBL 2PCF examples, with pairs/ and output/ directories

## Outputs

- Lognormal mock galaxy catalogues (real/simulated astronomical catalogues in the common CBL environment)
- 2PCF measurements on the mocks (Two Point Correlation class output)
- Covariance matrix (covariance_matrix.dat) / external error bars on the clustering measurement

## How to apply

First compile the relevant CBL library from the GitHub source with 'make lib*' (e.g. 'make libFUNC' builds libFUNC.so) and optionally 'make python' to obtain Python wrappers. Provide an input matter/galaxy power spectrum and set the cosmological parameters explicitly via a Cosmology constructor (e.g. Omega_B=0.049, Omega_M=0.319, Omega_DE=0.681, sigma8=0.83, h=0.67) rather than relying on defaults. Use the CBL random-catalogue machinery to generate lognormal mock catalogues whose catalogue volume is computed from the survey geometry (e.g. 3.677700e+07 [Mpc/h]^3 for a 1/8-sphere HOD catalogue). Measure the 2PCF on the mocks and compare the resulting covariance/error bars against internal sub-sampling estimates (jackknife deleting one sub-region at a time; bootstrap drawing sub-regions with replacement). Rationale: CBL is designed around a common environment for handling extragalactic catalogues, so mocks generated with the same classes are directly comparable to the data-side clustering analysis.

## Related tools

- **Cosmo Bologna Lib** (Core library: random-catalogue machinery to generate lognormal mocks from an input power spectrum, and Two Point Correlation classes to measure clustering on them) — https://github.com/federicomarulli/CosmoBolognaLib
- **C++** (Language in which CBL is fully written; used to drive mock generation directly)
- **Python** (Alternative driver language via the CBL Python wrappers built with 'make python')

## Examples

```
make libFUNC && make python
```

## Evaluation signals

- Generated mock catalogue volume is reported and matches the survey geometry (e.g. ~3.677700e+07 [Mpc/h]^3 for a 1/8-sphere HOD catalogue)
- 2PCF measured on the mocks is consistent with the input power spectrum used for the lognormal realisation
- Covariance matrix from the mocks is square, symmetric, positive semi-definite, with finite, non-negative diagonal errors
- Mock-based error bars agree with independent jackknife/bootstrap covariance estimates on the same catalogue within expected statistical scatter

## Limitations

- Only a subset of the CBL has a Python wrapper; some mock-generation functionality may require C++
- No fixed version/release identifier was given for the CBL download in the paper (only a .tar archive at a personal institutional URL); pin a GitHub release yourself
- The 2PCF example depends on local input files (cat.dat, random/, pairs/, output/ directories) that are not hosted, so paths must be recreated locally
- Cosmological parameters in examples default unless set explicitly via a different Cosmology constructor; parameter-setting details are deferred to example codes on the CBL webpage

## Evidence

- [intro] a large set of methods are provided to construct random catalogues, to estimate errors and to extract cosmological constraints from clustering analyses: [evidence span withheld — anchor status: not_found]
