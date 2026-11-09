---
name: effective-halo-bias-parameterisation
description: Use when modelling or interpreting halo/galaxy clustering at a given
  redshift and you need the effective halo bias b(z) from published parameterisations,
  or when comparing bias prescriptions against halo number densities dn/dM.
when_to_use_negative:
- The task is measuring an observed 2PCF from a galaxy catalogue rather than modelling
  halo bias from a parameterisation
- Only a subset of CBL has a Python wrapper, so if the required bias/mass-function
  function is not wrapped, working purely in Python is the wrong route — use C++
- The cosmology differs from the fixed set (Omega_B=0.049, Omega_M=0.319, Omega_DE=0.681,
  sigma8=0.83, h=0.67) without deliberately re-setting the constructor parameters,
  since default cosmological parameters are used when the object is created
edam_operation: http://edamontology.org/operation_2426
tools:
- name: Cosmo Bologna Lib
  role: Provides the Cosmology class and modelling classes that implement the halo
    bias parameterisations (Eq. 3) and mass functions (Eqs. 1-2) using GSL-based transfer
    functions
  repo: https://github.com/federicomarulli/CosmoBolognaLib
- name: C++
  role: Primary language for CBL; used when the Python wrapper does not cover the
    required bias/mass-function functions
- name: Python
  role: Alternative interface via the CBL Python wrappers (compiled with 'make python')
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2016.01.005
    title: 'CosmoBolognaLib: C++ libraries for cosmological calculations'
  source_content_hash: sha256:82c39418631045ad46e1e23be736a987f32c72054e57531e4fd959cb0a33e422
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
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# effective-halo-bias-parameterisation

## Summary

Evaluate the effective halo bias b(z) using the Sheth-Tormen 1999, Sheth 2001 and Tinker 2010 parameterisations implemented in the CosmoBolognaLib (CBL) modelling classes, at a fixed cosmology (Omega_B=0.049, Omega_M=0.319, Omega_DE=0.681, sigma8=0.83, h=0.67). It supports clustering analyses that weight halo samples by their expected bias.

## When to use

Use when modelling or interpreting halo/galaxy clustering at a given redshift and you need the effective halo bias b(z) from published parameterisations, or when comparing bias prescriptions against halo number densities dn/dM.

## When NOT to use

- The task is measuring an observed 2PCF from a galaxy catalogue rather than modelling halo bias from a parameterisation
- Only a subset of CBL has a Python wrapper, so if the required bias/mass-function function is not wrapped, working purely in Python is the wrong route — use C++
- The cosmology differs from the fixed set (Omega_B=0.049, Omega_M=0.319, Omega_DE=0.681, sigma8=0.83, h=0.67) without deliberately re-setting the constructor parameters, since default cosmological parameters are used when the object is created

## Inputs

- cosmological parameter set (Omega_B, Omega_M, Omega_DE, sigma8, h)
- grid of halo masses M
- redshift z at which b(z) is evaluated
- choice of bias parameterisation (Sheth-Tormen 1999, Sheth 2001, Tinker 2010)

## Outputs

- effective halo bias b(z) per prescription (e.g. bias column in halo_mass_function_bias.csv)
- optionally dn/dM per mass function model, one column per model in the CSV

## How to apply

Instantiate the CBL cosmobl::Cosmology object (in C++ or via the Python wrapper) and set the fixed parameters Omega_B=0.049, Omega_M=0.319, Omega_DE=0.681, sigma8=0.83, h=0.67. Compute the variance of the linear density field sigma(M) on a grid of halo masses using the GSL-based transfer function machinery in the CBL Cosmology module. Evaluate b(z) from Eq. 3 for each of the three bias prescriptions (Sheth-Tormen 1999, Sheth 2001, Tinker 2010) at the same cosmology and redshift. Optionally compute dn/dM from Eqs. 1-2 for the nine mass function models (Press-Schechter, Sheth-Tormen, Jenkins, Warren, Shen, Reed, Pan, Tinker, Angulo) alongside, and write both to a CSV (one column per model/prescription). Validate by checking the values match the reference CBL built-in model outputs and fall in physically sensible ranges for the fixed cosmology.

## Related tools

- **Cosmo Bologna Lib** (Provides the Cosmology class and modelling classes that implement the halo bias parameterisations (Eq. 3) and mass functions (Eqs. 1-2) using GSL-based transfer functions) — https://github.com/federicomarulli/CosmoBolognaLib
- **C++** (Primary language for CBL; used when the Python wrapper does not cover the required bias/mass-function functions)
- **Python** (Alternative interface via the CBL Python wrappers (compiled with 'make python'))

## Examples

```
cosm.set_OmegaB(0.049); cosm.set_OmegaM(0.319); cosm.set_OmegaDE(0.681); cosm.set_sigma8(0.83); cosm.set_hh(0.67)
```

## Evaluation signals

- Computed b(z) values for each prescription match the reference CBL built-in model outputs (output_matches_reference)
- b(z) values fall in physically sensible ranges for the fixed cosmology (Omega_M=0.319, sigma8=0.83, h=0.67)
- Script runs without error after installing external libraries (GSL, FFTW, OpenMP) and compiling with 'make'
- CSV output contains one column per bias prescription and one per mass function model, with no missing values on the mass grid
- sigma(M) values reproduce the GSL-based transfer function machinery's expected behaviour before entering the bias formula

## Limitations

- Only a subset of the CBL has currently a Python wrapper; full coverage was deferred to a future version
- No fixed version/release identifier is given for the CBL download (only a .tar archive), and the download location was a personal institutional webpage rather than an archival repository with a DOI
- Cosmological parameters in the examples rely on defaults; parameter-setting details are deferred to external example codes on the CBL webpage

## Evidence

- [intro] CBL is fully written in C++ with wrapping for scripting languages: [evidence span withheld — anchor status: not_found]
