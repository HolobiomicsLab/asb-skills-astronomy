---
name: reduced-3pcf-normalization
description: Use when measuring higher-order clustering of a simulated or observed
  galaxy/halo catalogue, specifically when ζ and the reduced 3PCF Q are needed as
  functions of triangle opening angle θ at fixed sides r12, r13.
when_to_use_negative:
- The analysis only requires two-point statistics — CBL's optimised 2PCF chain-mesh
  pair-counting pipeline is the appropriate path, not the 3PCF estimator
- The catalogue cannot be accompanied by a random catalogue, since the Szapudi-Szalay
  estimator requires DDD, DDR, DRR and RRR triplet counts
- Full Python-wrapper coverage of a specific CBL function is assumed — only a subset
  of CBL has a Python wrapper, so a C++ implementation may be required
tools:
- name: Cosmo Bologna Lib
  role: Provides the Szapudi-Szalay 3PCF estimator, parallel chain-mesh triplet counting,
    random catalogue construction, and reduced 3PCF Q normalization
  repo: https://github.com/federicomarulli/CosmoBolognaLib
- name: C++
  role: Primary language in which CBL is fully written; used to build the 3PCF measurement
    code when the Python wrapper is insufficient
- name: GSL
  role: External numerical library required by CBL; since V3.0 Numerical Recipes functions
    were fully substituted by GSL
- name: Python
  role: Optional high-level wrapper for driving CBL catalogue handling and clustering
    measurements
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2016.01.005
    title: 'CosmoBolognaLib: C++ libraries for cosmological calculations'
  source_content_hash: sha256:413913936e4d06888d4aadd8f6fade81d72c4e842ce157e7d4a33a9c63bc0127
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# reduced-3pcf-normalization

## Summary

Measure the connected three-point correlation function ζ(r12, r23, r31) of a galaxy or halo catalogue with the Szapudi-Szalay (1998) estimator implemented in CosmoBolognaLib, then normalize it into the reduced 3PCF Q = ζ/(ξ12ξ23 + ξ23ξ31 + ξ31ξ12), fixing two triangle sides and binning in the opening angle θ.

## When to use

Use when measuring higher-order clustering of a simulated or observed galaxy/halo catalogue, specifically when ζ and the reduced 3PCF Q are needed as functions of triangle opening angle θ at fixed sides r12, r13.

## When NOT to use

- The analysis only requires two-point statistics — CBL's optimised 2PCF chain-mesh pair-counting pipeline is the appropriate path, not the 3PCF estimator
- The catalogue cannot be accompanied by a random catalogue, since the Szapudi-Szalay estimator requires DDD, DDR, DRR and RRR triplet counts
- Full Python-wrapper coverage of a specific CBL function is assumed — only a subset of CBL has a Python wrapper, so a C++ implementation may be required

## Inputs

- Simulated halo or galaxy catalogue loadable by CosmoBolognaLib catalogue classes (e.g. ASCII catalogue file such as cat.dat)
- Corresponding random catalogue (constructed with CBL random-catalogue methods)
- Two-point correlation function measurements ξ for the triangle sides r12, r23, r31
- Bin configuration: fixed triangle sides and opening-angle θ bins

## Outputs

- Table of ζ(r12, r23, r31) values binned in the opening angle θ
- Table of reduced three-point correlation function Q(r12, r13, θ) values
- Intermediate triplet counts DDD, DDR, DRR, RRR

## How to apply

Load the data catalogue with CosmoBolognaLib catalogue-handling classes (C++, optionally via the Python wrapper) and construct a matching random catalogue using CBL's random-catalogue methods. Count data-data-data (DDD), data-data-random (DDR), data-random-random (DRR) and random-random-random (RRR) triplets at fixed triangle sides (r12, r23, r31) as a function of the opening angle θ, using CBL's parallel chain-mesh triplet-counting algorithm with GSL-backed numerics. Combine the four counts into ζ following the Szapudi and Szalay (1998) Eq. 11 estimator. Independently measure the two-point correlation functions ξ needed for the normalization, and compute the reduced 3PCF Q = ζ/(ξ12ξ23 + ξ23ξ31 + ξ31ξ12) as in Eq. 12. Write ζ and Q as a function of θ to a single output table.

## Related tools

- **Cosmo Bologna Lib** (Provides the Szapudi-Szalay 3PCF estimator, parallel chain-mesh triplet counting, random catalogue construction, and reduced 3PCF Q normalization) — https://github.com/federicomarulli/CosmoBolognaLib
- **C++** (Primary language in which CBL is fully written; used to build the 3PCF measurement code when the Python wrapper is insufficient)
- **GSL** (External numerical library required by CBL; since V3.0 Numerical Recipes functions were fully substituted by GSL)
- **Python** (Optional high-level wrapper for driving CBL catalogue handling and clustering measurements)

## Examples

```
make python
```

## Evaluation signals

- The output table contains both ζ and Q columns in the estimator-derived format, binned as a function of θ
- All ζ and Q values are finite and physically sensible across θ bins
- Reduced 3PCF Q is of order unity across the θ bins, as expected for gravitational clustering
- Triplet counts DDD, DDR, DRR and RRR are all non-zero and consistent between data and random catalogues at the fixed triangle sides
- Normalization reproduces Q = ζ/(ξ12ξ23 + ξ23ξ31 + ξ31ξ12) exactly when recomputed from the stored ζ and ξ values

## Limitations

- Only a subset of CBL has a Python wrapper; the 3PCF classes may require a C++ implementation
- The 2PCF/3PCF example workflow depends on local input files (cat.dat, random/, pairs/, output/ directories) that are not hosted with the paper
- No fixed version/release identifier or DOI/zenodo is given for the CBL download; only a .tar archive at a personal institutional URL, and the repository contains no changelog
- External dependencies (GSL, FFTW, OpenMP, Numerical Recipes 3rd ed.) must be installed before compiling

## Evidence

- [intro] optimised 2PCF/3PCF chain-mesh pair/triplet counting: [evidence span withheld — anchor status: not_found]
- [intro] CBL usable in C++ and wrapped scripting languages: [evidence span withheld — anchor status: not_found]
