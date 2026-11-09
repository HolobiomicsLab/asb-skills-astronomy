---
name: halo-mass-function-model-computation
description: Use when estimating halo number density dn/dM on a mass grid or comparing
  halo mass function and halo bias models at a fixed cosmology, e.g. for halo occupation
  or cluster-count modelling.
when_to_use_negative:
- When the Python wrapper is assumed to expose every CBL routine — only a subset of
  CBL functions currently has a Python wrapper, so some modelling calls may require
  C++
- When the task is measuring galaxy clustering statistics (2PCF/3PCF) from catalogues
  rather than modelling halo abundance and bias
- When a modified-gravity or dynamical dark-energy growth calculation is needed —
  that requires a Boltzmann-solver patch such as MGCAMB, not the CBL mass function
  module
tools:
- name: Cosmo Bologna Lib
  role: Provides the Cosmology class and modelling classes used to compute sigma(M),
    dn/dM for the mass function models and b(z) for the bias parameterisations
  repo: https://github.com/federicomarulli/CosmoBolognaLib
- name: C++
  role: Primary language in which the CBL libraries and halo mass function/bias routines
    are written and invoked
  repo: https://github.com/federicomarulli/CosmoBolognaLib
- name: Python
  role: Alternative interface via SWIG-compiled Python wrappers for calling the CBL
    Cosmology and modelling classes
  repo: https://github.com/federicomarulli/CosmoBolognaLib
- name: GSL
  role: External dependency backing the transfer function machinery used to compute
    the linear density field variance sigma(M)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2016.01.005
    title: 'CosmoBolognaLib: C++ libraries for cosmological calculations'
  source_content_hash: sha256:37a32a1b606d40ace5bfc4fd21d25863287999aa7e1e76da89074445481597ea
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/federicomarulli/CosmoBolognaLib
  tools:
  - C++
  - Cosmo Bologna Lib
  - GSL
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# halo-mass-function-model-computation

## Summary

Computes dark matter halo number densities dn/dM from multiple halo mass function parameterisations (Press-Schechter, Sheth-Tormen, Jenkins, Warren, Shen, Reed, Pan, Tinker, Angulo) and effective halo bias b(z) (Sheth-Tormen 1999, Sheth 2001, Tinker 2010) using the CosmoBolognaLib C++/Python cosmology libraries.

## When to use

Use when estimating halo number density dn/dM on a mass grid or comparing halo mass function and halo bias models at a fixed cosmology, e.g. for halo occupation or cluster-count modelling.

## When NOT to use

- When the Python wrapper is assumed to expose every CBL routine — only a subset of CBL functions currently has a Python wrapper, so some modelling calls may require C++
- When the task is measuring galaxy clustering statistics (2PCF/3PCF) from catalogues rather than modelling halo abundance and bias
- When a modified-gravity or dynamical dark-energy growth calculation is needed — that requires a Boltzmann-solver patch such as MGCAMB, not the CBL mass function module

## Inputs

- Cosmological parameter set (Omega_B=0.049, Omega_M=0.319, Omega_DE=0.681, sigma8=0.83, h=0.67 in the reference example)
- Grid of halo masses
- Choice of mass function model(s): Press-Schechter, Sheth-Tormen, Jenkins, Warren, Shen, Reed, Pan, Tinker, Angulo
- Choice of bias parameterisation: Sheth-Tormen 1999, Sheth 2001, Tinker 2010

## Outputs

- dn/dM values per mass function model (one column per model) written to halo_mass_function_bias.csv
- Effective halo bias b(z) values per prescription (one column per bias parameterisation)

## How to apply

Instantiate a cosmobl::Cosmology object in C++ (or via the Python wrapper) and set the cosmological parameters explicitly, e.g. Omega_B=0.049, Omega_M=0.319, Omega_DE=0.681, sigma8=0.83, h=0.67, since defaults are used otherwise. On a grid of halo masses, compute the variance of the linear density field sigma(M) with the GSL-based transfer function machinery in the CBL Cosmology module (Numerical Recipes having been fully substituted by GSL in V3.0). Evaluate dn/dM for each mass function model in the CBL modelling classes, then evaluate the effective halo bias b(z) from Eq. 3 for the Sheth-Tormen 1999, Sheth 2001 and Tinker 2010 parameterisations. Decision point: choose which of the nine mass function models to tabulate based on the analysis need; write one column per model and per bias prescription to a CSV such as halo_mass_function_bias.csv. Compile the library first with 'make' (or 'make lib*' for individual libraries and 'make python' for wrappers) after installing GSL, FFTW and OpenMP.

## Related tools

- **Cosmo Bologna Lib** (Provides the Cosmology class and modelling classes used to compute sigma(M), dn/dM for the mass function models and b(z) for the bias parameterisations) — https://github.com/federicomarulli/CosmoBolognaLib
- **C++** (Primary language in which the CBL libraries and halo mass function/bias routines are written and invoked) — https://github.com/federicomarulli/CosmoBolognaLib
- **Python** (Alternative interface via SWIG-compiled Python wrappers for calling the CBL Cosmology and modelling classes) — https://github.com/federicomarulli/CosmoBolognaLib
- **GSL** (External dependency backing the transfer function machinery used to compute the linear density field variance sigma(M))

## Examples

```
cosm.set_OmegaM(0.319); cosm.set_OmegaDE(0.681); cosm.set_sigma8(0.83); cosm.set_hh(0.67)
```

## Evaluation signals

- Script runs without error after compiling CBL ('make' succeeds with GSL, FFTW, OpenMP installed)
- Computed dn/dM values for each model match the reference CBL built-in model outputs (output_matches_reference)
- Computed b(z) values for each bias prescription fall in physically sensible ranges for the fixed cosmology (Omega_M=0.319, sigma8=0.83, h=0.67)
- Output file halo_mass_function_bias.csv contains one column per mass function model and one per bias prescription
- Parameter setters return the intended values (e.g. cosm.set_OmegaM(0.319); cosm.set_sigma8(0.83)) rather than silent CBL defaults

## Limitations

- Only a subset of the CBL has a Python wrapper; full coverage deferred to a future version of the libraries
- No fixed version/release identifier or archival DOI is given for the download; only a .tar archive at a personal institutional URL
- Cosmological parameters in the examples rely on defaults unless set explicitly via a different constructor
- Compilation depends on external libraries (GSL, FFTW, OpenMP, formerly Numerical Recipes) being installed; MAC OS needs 'make SYS=`MAC`'

## Evidence
