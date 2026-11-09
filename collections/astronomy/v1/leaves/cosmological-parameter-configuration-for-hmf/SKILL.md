---
name: cosmological-parameter-configuration-for-hmf
description: Use when setting up or varying the cosmology (Omega_c, Omega_Lambda,
  sigma_8, spectral index, transfer function source, growth factor) underlying an
  hmf/HMFcalc halo mass function calculation.
when_to_use_negative:
- The analysis targets the halo model (2-halo/1-halo terms) rather than the dark matter
  halo mass function — use halomod/TheHaloMod instead
- The transfer function cannot be produced by or formatted as CAMB output and Eisenstein
  & Hu extrapolation is insufficient
- You need to regenerate the entire pipeline from scratch on every parameter change
  and cannot exploit the cascading update mechanism
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: hmf
  role: Python engine that performs the HMF calculation and exposes the update() method
    for parameter changes
  repo: https://github.com/halomod/hmf
- name: HMFcalc
  role: Web frontend for interactively exploring cosmological parameters, redshift
    and fitting function effects on the predicted HMF
  repo: http://hmf.icrar.org
- name: CAMB
  role: Computes the transfer functions that seed the unnormalised power spectrum
    P0(k)=k^n*T^2
  repo: https://github.com/joezuntz/pycamb
- name: Django
  role: Web framework underlying the HMFcalc frontend for parameter configuration
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2013.11.001
    title: 'HMFcalc: An online tool for calculating dark matter halo mass functions'
  source_content_hash: sha256:461c6ff6b5e3ebc34690668555cb257617af4d5fe17efd451138097f10a5ab74
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/halomod/hmf
  tools:
  - CAMB
  - Django
  - HMFcalc
  - Python
  - hmf
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# cosmological-parameter-configuration-for-hmf

## Summary

Configure the cosmological and transfer-function parameters that feed the hmf halo mass function calculation — instantiating a MassFunction with default cosmology and sigma_8 normalisation, and modifying individual parameters via the cascading update() method so only dependent quantities are recomputed.

## When to use

Use when setting up or varying the cosmology (Omega_c, Omega_Lambda, sigma_8, spectral index, transfer function source, growth factor) underlying an hmf/HMFcalc halo mass function calculation.

## When NOT to use

- The analysis targets the halo model (2-halo/1-halo terms) rather than the dark matter halo mass function — use halomod/TheHaloMod instead
- The transfer function cannot be produced by or formatted as CAMB output and Eisenstein & Hu extrapolation is insufficient
- You need to regenerate the entire pipeline from scratch on every parameter change and cannot exploit the cascading update mechanism

## Inputs

- Cosmological parameter set (Omega_c, Omega_Lambda, sigma_8, spectral index n, redshift z)
- Transfer function from CAMB or a CAMB-format file
- Mass grid in h^-1 M_sun (e.g. 10^10-10^16)
- Optional WDM model selection

## Outputs

- Normalised linear power spectrum P(k) and mass variance sigma(M,z)
- HMF quantities (dndlnm, ngtm) computed under the configured cosmology
- Updated parameter state of the MassFunction object after each update() call

## How to apply

Instantiate a MassFunction (Perturbations) object with default cosmological parameters and sigma_8 normalisation over the desired mass grid (e.g. 10^10-10^16 h^-1 M_sun at z=0). Obtain the transfer function via CAMB (or a CAMB-format file, optionally extrapolated with extrapolate_with_eh=True using Eisenstein & Hu's fitting formula), interpolate it with linear splines in log-space, and form P0(k)=k^n*T^2 before normalising with sigma_8. To change any single parameter (cosmology, redshift, or fitting function), call update() with the relevant keyword argument — the cascading dependency tracker deletes only directly dependent quantities, cutting processing time by ~50% when each parameter changes once and >94% when only redshift changes. Growth factors are obtained by solving the growth ODE as a coupled first-order system with D(a) ~ 2*a_eq/3 and D'(a) ~ 0 as a -> 0, and a WDM model applies the T_WDM(k) correction to the power spectrum.

## Related tools

- **hmf** (Python engine that performs the HMF calculation and exposes the update() method for parameter changes) — https://github.com/halomod/hmf
- **HMFcalc** (Web frontend for interactively exploring cosmological parameters, redshift and fitting function effects on the predicted HMF) — http://hmf.icrar.org
- **CAMB** (Computes the transfer functions that seed the unnormalised power spectrum P0(k)=k^n*T^2) — https://github.com/joezuntz/pycamb
- **Django** (Web framework underlying the HMFcalc frontend for parameter configuration)

## Examples

```
from hmf import MassFunction; mf = MassFunction(Mmin=10, Mmax=16, hmf_model='ST'); mf.update(z=1.0)
```

## Evaluation signals

- sigma_8 normalisation applied: P(k) rescaled by (sigma_8/sigma_8_unnormalised)^2 relative to P0(k)=k^n*T^2
- After a single update() call, only directly dependent quantities are deleted/recalculated — e.g. changing only redshift recomputes growth and sigma(M,z) but not the transfer function
- dndlnm values are recomputed consistently with the fitting-function dictionary entry selected (e.g. 'ST')
- Transfer function interpolation uses linear splines in log-space between k-bounds; cumulative HMF uses romberg integration
- Runtime check: cascading method reduces processing time by ~50% when each parameter is changed once versus full recomputation

## Limitations

- pycamb is outdated: 'This code is out of data and no longer works with recent versions of camb. Since camb itself now ships with python bindings you should use those instead.'
- Transfer function files must be either produced by CAMB or in CAMB format; high-k coverage may require extrapolate_with_eh=True
- The 2013 paper's parameter ranges reflect its era — future work noted plans to broaden the range of dark matter and dark energy models considered

## Evidence

- [methods] Any variable can be updated by calling the update() method with relevant keyword arguments; quantities that depend directly on the updated variables are deleted: [evidence span withheld — anchor status: not_found]
- [methods] the cascading method reduces processing time by 50% in tests where each available parameter is changed one time: "the cascading method reduces processing time by 50% in tests where each available parameter is changed one time"
- [methods] If a filename is provided, the relevant file will be read and used as the transfer function. This file must be either produced by CAMB or in CAMB format.: "If a filename is provided, the relevant file will be read and used as the transfer function. This file must be either produced by CAMB or in CAMB format."
- [intro] We use the public Code for Anisotropies in the Microwave Background (CAMB) (Lewis et al., 2000) to compute our transfer functions.: "We use the public Code for Anisotropies in the Microwave Background (CAMB) (Lewis et al., 2000) to compute our transfer functions."
- [readme] hmf is a python application that provides a flexible and simple way to calculate the Halo Mass Function for a range of varying parameters: "``hmf`` is a python application that provides a flexible and simple way to calculate the
Halo Mass Function for a range of varying parameters."
