---
name: cosmological-growth-rate-evaluation
description: Use when the goal is a numerical value of the linear growth rate fσ8
  at a specific redshift under a specified cosmology, and an external Boltzmann solver
  (CAMB) may be used as the computation backend.
when_to_use_negative:
- The task is measuring galaxy clustering statistics (2PCF/3PCF) or building random
  catalogues rather than evaluating the theoretical growth rate fσ8 at a redshift
- The pipeline is Python-only and needs full CBL coverage — the Python wrapper covers
  only a subset of CBL functions
- A version-pinned, archival (DOI/Zenodo) software release is required — the article
  only references a .tar archive at a personal institutional URL with no fixed release
  identifier
tools:
- name: CosmoBolognaLib
  role: Provides the cosmobl::Cosmology class whose fsigma8(z, method) method is called
    at z=1
  repo: https://github.com/federicomarulli/CosmoBolognaLib
- name: CAMB
  role: External Boltzmann solver dispatched by the fsigma8 method argument to perform
    the underlying computation
- name: GSL
  role: Required external library that must be present to build the CBL libraries
    (replaced Numerical Recipes from V3.0)
- name: C++
  role: Language in which the example program invoking cosmobl::Cosmology is written
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2016.01.005
    title: 'CosmoBolognaLib: C++ libraries for cosmological calculations'
  source_content_hash: sha256:9c7b0555257f6a864916d68e42087d4150acfee1dbf606b1846347af96a13111
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/federicomarulli/CosmoBolognaLib
  tools:
  - C++
  - CAMB
  - Cosmo Bologna Lib
  - CosmoBolognaLib
  - GSL
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cosmological-growth-rate-evaluation

## Summary

Estimate the cosmological growth rate fσ8 at a given redshift (e.g. z=1) by constructing a cosmobl::Cosmology object in C++ and dispatching the computation to the external CAMB Boltzmann solver via the fsigma8(z, method) method of CosmoBolognaLib.

## When to use

Use when the goal is a numerical value of the linear growth rate fσ8 at a specific redshift under a specified cosmology, and an external Boltzmann solver (CAMB) may be used as the computation backend.

## When NOT to use

- The task is measuring galaxy clustering statistics (2PCF/3PCF) or building random catalogues rather than evaluating the theoretical growth rate fσ8 at a redshift
- The pipeline is Python-only and needs full CBL coverage — the Python wrapper covers only a subset of CBL functions
- A version-pinned, archival (DOI/Zenodo) software release is required — the article only references a .tar archive at a personal institutional URL with no fixed release identifier

## Inputs

- Compiled CosmoBolognaLib C++ libraries (built with GSL, FFTW, OpenMP available)
- C++ example program constructing a cosmobl::Cosmology object (Listing 1 of Appendix B.1)
- Cosmological parameter set (defaults at object creation, or explicit Planck-like values e.g. Omega_M=0.319, sigma8=0.83, h=0.67)
- External CAMB Boltzmann solver installation

## Outputs

- Printed value of f*sigma8 at z=1
- Single-float output file fsigma8_z1_camb.txt

## How to apply

First build the CosmoBolognaLib C++ libraries from source after confirming the external dependencies (GSL, FFTW, OpenMP; Numerical Recipes was substituted by GSL from V3.0), by running 'make' in the Cosmo_Bologna_Lib/ folder. In the C++ example program, instantiate a cosmobl::Cosmology object — at creation it uses default (Planck-like) cosmological parameters, or set them explicitly with setters such as set_OmegaB(0.049), set_OmegaM(0.319), set_OmegaDE(0.681), set_sigma8(0.83), set_hh(0.67). Call the fsigma8 method at redshift z=1 passing a method argument that selects the external CAMB Boltzmann code as the backend. Capture the printed f*sigma8 value and write it as a single numeric entry (e.g. to fsigma8_z1_camb.txt) for downstream validation.

## Related tools

- **CosmoBolognaLib** (Provides the cosmobl::Cosmology class whose fsigma8(z, method) method is called at z=1) — https://github.com/federicomarulli/CosmoBolognaLib
- **CAMB** (External Boltzmann solver dispatched by the fsigma8 method argument to perform the underlying computation)
- **GSL** (Required external library that must be present to build the CBL libraries (replaced Numerical Recipes from V3.0))
- **C++** (Language in which the example program invoking cosmobl::Cosmology is written)

## Examples

```
cosmobl::Cosmology cosm; double fs8 = cosm.fsigma8(1., "CAMB");
```

## Evaluation signals

- The output file fsigma8_z1_camb.txt exists and parses as a single float
- The f*sigma8 value at z=1 lies in the physically expected range ~0.4–0.6 for the default Planck cosmology
- The computed value is consistent with the value printed by the Appendix B.1 example code
- The libraries compiled successfully (e.g. 'make' in the Cosmo_Bologna_Lib/ folder completed, confirming GSL/FFTW/OpenMP availability)
- The method argument passed to fsigma8 selects CAMB, so the result reflects CAMB's Boltzmann-solver output rather than an internal CBL approximation

## Limitations

- When the Cosmology object is created, default cosmological parameters are used; parameter-setting details are deferred to external example codes on the CBL webpage
- Only a subset of the CBL has a Python wrapper; full coverage is deferred to a future version, so this workflow is C++-native
- No fixed version/release identifier is given for the CBL download and there is no archival (DOI/Zenodo) repository, so reproducibility depends on the git commit used
- The 2PCF-style examples depend on local input files (cat.dat, random/, pairs/, output/ directories) that are not described or hosted — this does not affect the fσ8 example but reflects example fragility

## Evidence
