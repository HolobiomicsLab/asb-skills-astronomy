---
name: redshift-space-power-spectrum-modelling
description: Use when reconstructing a model redshift-space 2PCF ξ(s) from a power
  spectrum with RSD, de-wiggled BAO wiggles, and Gaussian redshift errors, e.g. for
  BAO fitting with CosmoBolognaLib.
when_to_use_negative:
- The task is measuring ξ(s) from an observed galaxy catalogue (pair counting with
  the chain-mesh algorithm), not constructing a model power-spectrum-based 2PCF
- Only real-space clustering is required — the µ-dependent RSD terms and redshift-error
  damping σ are not needed
- The Python wrapper is required for the specific modelling function, since only a
  subset of CBL functions currently has a Python wrapper
edam_operation: http://edamontology.org/operation_0279
tools:
- name: CosmoBolognaLib
  role: C++ libraries providing the Cosmology class and the two-point-correlation
    modelling classes used to build P(k,µ), the µ-integrated multipoles, and the Fourier
    anti-transform to ξ(s)
  repo: https://github.com/federicomarulli/CosmoBolognaLib
- name: GSL
  role: GNU Scientific Library linked by CBL from V3.0 to replace Numerical Recipes;
    its quadrature routines perform the µ-integrations of Eqs. 20–22
- name: C++
  role: Language in which the modelling workflow is implemented and compiled (e.g.
    'make libFUNC' produces libFUNC.so)
- name: Python
  role: Alternative interface to CBL via wrappers ('make python'), though only a subset
    of CBL functions is wrapped
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ascom.2016.01.005
    title: 'CosmoBolognaLib: C++ libraries for cosmological calculations'
  source_content_hash: sha256:ed9b5c4b8ff70c5e36176c9f9080e4511d2fa7ea8fb94d7f098f94e9a91632f1
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
  edam_operation: http://edamontology.org/operation_0279
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# redshift-space-power-spectrum-modelling

## Summary

Model the angle-averaged two-point correlation function of cosmic tracers in redshift space by constructing the redshift-space power spectrum P(k,µ) (Eq. 16) with the linear growth rate f and Gaussian redshift-error damping σ, applying the de-wiggled BAO prescription (Eq. 23, Sigma_NL), and Fourier anti-transforming the µ-integrated multipoles (Eqs. 20–22) using CosmoBolognaLib modelling classes.

## When to use

Use when reconstructing a model redshift-space 2PCF ξ(s) from a power spectrum with RSD, de-wiggled BAO wiggles, and Gaussian redshift errors, e.g. for BAO fitting with CosmoBolognaLib.

## When NOT to use

- The task is measuring ξ(s) from an observed galaxy catalogue (pair counting with the chain-mesh algorithm), not constructing a model power-spectrum-based 2PCF
- Only real-space clustering is required — the µ-dependent RSD terms and redshift-error damping σ are not needed
- The Python wrapper is required for the specific modelling function, since only a subset of CBL functions currently has a Python wrapper

## Inputs

- CBL Cosmology object configured via setters (Omega_B, Omega_M, Omega_DE, sigma8, h)
- Real-space linear/non-linear dark matter power spectrum P_DM(k)
- Linear growth rate f of cosmic structures
- Damping parameters: σ (Gaussian redshift errors) and Sigma_NL (BAO wiggle damping)
- Compiled CBL modelling libraries (libFUNC.so and clustering modelling modules)

## Outputs

- Redshift-space power spectrum P(k,µ) (Eq. 16)
- µ-integrated monopole, quadrupole and hexadecapole terms (Eqs. 20–22)
- De-wiggled power spectrum (Eq. 23) with Sigma_NL damping
- model_2pcf.dat with s, ξ(s) columns over the chosen separation range

## How to apply

Build the relevant CBL modules from the source tree (e.g. 'make libFUNC' plus the modelling libraries), noting that from V3.0 GSL functions fully substitute Numerical Recipes, so the µ-integrations (Eqs. 20–22) use GSL quadrature. Instantiate a CBL Cosmology object for the test cosmology and set background parameters (Omega_B, Omega_M, Omega_DE, sigma8, h) via the class setters rather than constructor defaults, e.g. Omega_M=0.319, sigma8=0.83, h=0.67. Evaluate P(k,µ) (Eq. 16), combining the real-space linear/non-linear matter power spectrum with the redshift-space-distortion terms, where P_DM(k) is the dark matter power spectrum and f is the linear growth rate; σ enters as the damping scale describing Gaussian redshift errors. Construct the de-wiggled power spectrum (Eq. 23) by applying the Sigma_NL Gaussian damping to the BAO wiggles of the model P(k). Finally obtain the model ξ(s) by Fourier anti-transform of the de-wiggled, µ-integrated power spectrum, using the CBL two-point-correlation modelling classes improved from V4.1 onward, and write the s, ξ(s) columns to model_2pcf.dat.

## Related tools

- **CosmoBolognaLib** (C++ libraries providing the Cosmology class and the two-point-correlation modelling classes used to build P(k,µ), the µ-integrated multipoles, and the Fourier anti-transform to ξ(s)) — https://github.com/federicomarulli/CosmoBolognaLib
- **GSL** (GNU Scientific Library linked by CBL from V3.0 to replace Numerical Recipes; its quadrature routines perform the µ-integrations of Eqs. 20–22)
- **C++** (Language in which the modelling workflow is implemented and compiled (e.g. 'make libFUNC' produces libFUNC.so))
- **Python** (Alternative interface to CBL via wrappers ('make python'), though only a subset of CBL functions is wrapped)

## Examples

```
cosm.set_OmegaM(0.319); cosm.set_OmegaDE(0.681); cosm.set_sigma8(0.83); cosm.set_hh(0.67);
```

## Evaluation signals

- model_2pcf.dat contains the expected two columns (s, ξ(s)) over the chosen separation range with no NaN/empty entries
- The Cosmology object's parameters match the test cosmology set via setters (Omega_M=0.319, Omega_DE=0.681, sigma8=0.83, h=0.67), not constructor defaults
- P(k,µ) exhibits BAO wiggles damped as prescribed by Sigma_NL in the de-wiggled spectrum (Eq. 23)
- The Fourier anti-transform of the µ-integrated power spectrum recovers a smooth ξ(s) monopole consistent with the input P_DM(k) at large scales
- Compilation of the individual libraries succeeds (libFUNC.so produced) and the modelling classes come from V3.0+ (GSL-based) / V4.1+ (improved 2PCF modelling) versions

## Limitations

- Only a subset of the CBL functions currently has a Python wrapper; full coverage is deferred to a future version
- The 2PCF modelling examples depend on local input files and directories (cat.dat, random/, pairs/, output/) not described or hosted in the paper
- No fixed version/release identifier is given for the CBL download in the paper; version-specific behaviour (V3.0 GSL substitution, V4.1 modelling improvements) must be checked in the repository README/changelog
- When the Cosmology object is created with the default constructor, default cosmological parameters are used, which may silently differ from the intended test cosmology unless setters are called

## Evidence
