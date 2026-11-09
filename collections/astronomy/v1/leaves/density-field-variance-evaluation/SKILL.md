---
name: density-field-variance-evaluation
description: Use when estimating halo number density dn/dM or effective halo bias
  b(z) from dark matter halo mass function models (Press-Schechter, Sheth-Tormen,
  Tinker, etc.) and you need sigma(M) at a fixed cosmology.
when_to_use_negative:
- The analysis requires only galaxy clustering statistics (2PCF/3PCF measurements)
  rather than halo mass function modelling — use CBL's catalogue-handling and correlation-function
  classes instead.
- 'A Python-only workflow is required with full function coverage: the CBL Python
  wrapper covers only a subset of the libraries, so some sigma(M)/mass function calls
  must be done in C++.'
- The target is modified-gravity Boltzmann-solver computations (e.g. mu-gamma parametrisations)
  rather than halo abundance modelling — MGCAMB/CAMB is the wrong-scope sibling tool.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3452
- http://edamontology.org/topic_0153
tools:
- name: Cosmo Bologna Lib
  role: Provides the cosmobl::Cosmology class and GSL-based transfer function machinery
    used to compute sigma(M), dn/dM and b(z) for the implemented mass function and
    bias models
  repo: https://github.com/federicomarulli/CosmoBolognaLib
- name: C++
  role: Primary language for directly calling CBL Cosmology-class methods to evaluate
    sigma(M) and the mass function models
  repo: https://github.com/federicomarulli/CosmoBolognaLib
- name: Python
  role: High-level scripting access to the (partial) SWIG-wrapped subset of CBL functions
    for evaluating sigma(M) and dn/dM
  repo: https://github.com/federicomarulli/CosmoBolognaLib
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2016.01.005
    title: 'CosmoBolognaLib: C++ libraries for cosmological calculations'
  source_content_hash: sha256:95cd02d0180c7ee073bef1c7f27503562d60e9b01d84c3a183ca1f3d6f2ef811
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/federicomarulli/CosmoBolognaLib
  tools:
  - C++
  - Cosmo Bologna Lib
  - Python
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3452
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# density-field-variance-evaluation

## Summary

Compute the variance of the linear density field, sigma(M), on a grid of halo masses using the GSL-based transfer function machinery of the CosmoBolognaLib Cosmology module, as the ingredient needed for halo mass function dn/dM and effective halo bias b(z) estimates.

## When to use

Use when estimating halo number density dn/dM or effective halo bias b(z) from dark matter halo mass function models (Press-Schechter, Sheth-Tormen, Tinker, etc.) and you need sigma(M) at a fixed cosmology.

## When NOT to use

- The analysis requires only galaxy clustering statistics (2PCF/3PCF measurements) rather than halo mass function modelling — use CBL's catalogue-handling and correlation-function classes instead.
- A Python-only workflow is required with full function coverage: the CBL Python wrapper covers only a subset of the libraries, so some sigma(M)/mass function calls must be done in C++.
- The target is modified-gravity Boltzmann-solver computations (e.g. mu-gamma parametrisations) rather than halo abundance modelling — MGCAMB/CAMB is the wrong-scope sibling tool.

## Inputs

- Fixed cosmological parameter set (Omega_B=0.049, Omega_M=0.319, Omega_DE=0.681, sigma8=0.83, h=0.67)
- Grid of dark matter halo masses
- Redshift values z at which to evaluate sigma(M), dn/dM and b(z)
- Choice of halo mass function model (Press-Schechter, Sheth-Tormen, Jenkins, Warren, Shen, Reed, Pan, Tinker, Angulo) and bias parameterisation (Sheth-Tormen 1999, Sheth 2001, Tinker 2010)

## Outputs

- sigma(M) values of the linear density field variance per halo mass
- dn/dM halo number density values, one column per mass function model
- Effective halo bias b(z) values, one column per bias prescription
- CSV file halo_mass_function_bias.csv combining the dn/dM and b(z) columns

## How to apply

Instantiate the CBL cosmobl::Cosmology object in C++ or via its Python wrapper and fix the cosmological parameters Omega_B=0.049, Omega_M=0.319, Omega_DE=0.681, sigma8=0.83, h=0.67. On a grid of halo masses, evaluate the variance of the linear density field sigma(M) using the GSL-based transfer function machinery in the CBL Cosmology module. Feed sigma(M) into dn/dM (Eqs. 1-2) for the nine implemented mass function models (Press-Schechter, Sheth-Tormen, Jenkins, Warren, Shen, Reed, Pan, Tinker, Angulo) and into the effective halo bias b(z) (Eq. 3) for the Sheth-Tormen 1999, Sheth 2001 and Tinker 2010 parameterisations. Write dn/dM (one column per model) and b(z) (one column per bias prescription) to halo_mass_function_bias.csv. Note that CBL requires GSL, FFTW and OpenMP to be installed and the libraries compiled with 'make' (or 'make lib*' for individual libraries, 'make python' for the SWIG wrappers).

## Related tools

- **Cosmo Bologna Lib** (Provides the cosmobl::Cosmology class and GSL-based transfer function machinery used to compute sigma(M), dn/dM and b(z) for the implemented mass function and bias models) — https://github.com/federicomarulli/CosmoBolognaLib
- **C++** (Primary language for directly calling CBL Cosmology-class methods to evaluate sigma(M) and the mass function models) — https://github.com/federicomarulli/CosmoBolognaLib
- **Python** (High-level scripting access to the (partial) SWIG-wrapped subset of CBL functions for evaluating sigma(M) and dn/dM) — https://github.com/federicomarulli/CosmoBolognaLib

## Examples

```
cosm.set_OmegaM(0.319); cosm.set_sigma8(0.83); cosm.set_hh(0.67); // then compute sigma(M) on the halo-mass grid via the CBL Cosmology class
```

## Evaluation signals

- Script runs end-to-end without error after compiling CBL ('make' / 'make python') with GSL, FFTW and OpenMP installed
- Computed dn/dM values for each of the nine models match the reference CBL built-in model outputs (output_matches_reference)
- Computed b(z) values for the Sheth-Tormen 1999, Sheth 2001 and Tinker 2010 prescriptions match the reference implementations
- Output dn/dM and b(z) values fall in physically sensible ranges for the fixed cosmology (Omega_M=0.319, sigma8=0.83, h=0.67)
- halo_mass_function_bias.csv contains one column per mass function model and one column per bias prescription as specified

## Limitations

- Only a subset of the CBL functions has a Python wrapper; full coverage is deferred to a future version of the libraries, so some calls must be made in C++.
- No fixed version/release identifier is given for the CBL download; the .tar archive is hosted on a personal institutional webpage rather than an archival repository (no DOI/zenodo).
- Cosmological parameters in the examples rely on defaults unless set explicitly; parameter-setting details are deferred to external example codes on the CBL webpage.
- The GitHub repository contains no changelog, making version provenance of model implementations hard to trace.

## Evidence

- [results] the nine mass function models (Press-Schechter, Sheth-Tormen, Jenkins, Warren, Shen, Reed, Pan, Tinker, Angulo) implemented in the CBL modelling classes: [evidence span withheld — anchor status: not_found]
- [intro] The CBL are fully written in C++ and can be included in C++ codes or in high-level scripting languages through wrapping: [evidence span withheld — anchor status: not_found]
