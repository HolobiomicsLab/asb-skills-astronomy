---
name: top-hat-mass-variance-integration
description: Use when you need the differential or cumulative dark matter halo mass
  function (dndlnm, ngtm) from a linear power spectrum normalised to sigma_8 with
  a top-hat window function, as implemented in the hmf/HMFcalc framework.
when_to_use_negative:
- You need CMB power spectra, the age of the universe, or angular diameter distances
  rather than halo mass functions — call pycamb.camb, pycamb.age or pycamb.angular_diameter
  directly instead of driving hmf.
- Your transfer function comes from a non-CAMB source or a file not in CAMB format
  — hmf will not read it, and Eisenstein & Hu extrapolation (extrapolate_with_eh=True)
  only applies to high-k CAMB transfer functions.
- You need halo abundances for dark matter models outside the CDM scope of the paper
  (e.g. bespoke WDM models) that the current hmf/HMFcalc fitting-function set does
  not cover.
tools:
- name: hmf
  role: Python engine that performs the sigma_8-normalised power-spectrum calculation,
    top-hat mass-variance integration, and HMF evaluation (dndlnm, ngtm)
  repo: https://github.com/halomod/hmf
- name: CAMB
  role: Computes the transfer functions required for the linear power spectrum underlying
    the HMF calculation
  repo: http://camb.info
- name: pycamb
  role: Python bindings that let hmf call CAMB's transfer-function routines (pycamb.transfers
    returns k, T_i(k,z), sigma8(z))
  repo: https://github.com/joezuntz/pycamb
- name: HMFcalc
  role: Web frontend (http://hmf.icrar.org) to the hmf engine for interactive HMF
    exploration
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2013.11.001
    title: 'HMFcalc: An online tool for calculating dark matter halo mass functions'
  source_content_hash: sha256:c47ab28884d479aa40b15a899a853528838b596c4fb12f932557a7e49dc98ada
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/halomod/hmf
  tools:
  - CAMB
  - HMFcalc
  - Python
  - hmf
  - pycamb
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# top-hat-mass-variance-integration

## Summary

Computes the dark matter halo mass function (dn/dlnM and ngtm) by normalising the linear power spectrum to sigma_8 (mass variance on the 8 h^-1 Mpc scale at z=0 with a top-hat window), applying the linear growth factor, and integrating the mass variance over a top-hat filter to evaluate fitting functions such as Tinker (2008) via the hmf engine.

## When to use

Use when you need the differential or cumulative dark matter halo mass function (dndlnm, ngtm) from a linear power spectrum normalised to sigma_8 with a top-hat window function, as implemented in the hmf/HMFcalc framework.

## When NOT to use

- You need CMB power spectra, the age of the universe, or angular diameter distances rather than halo mass functions — call pycamb.camb, pycamb.age or pycamb.angular_diameter directly instead of driving hmf.
- Your transfer function comes from a non-CAMB source or a file not in CAMB format — hmf will not read it, and Eisenstein & Hu extrapolation (extrapolate_with_eh=True) only applies to high-k CAMB transfer functions.
- You need halo abundances for dark matter models outside the CDM scope of the paper (e.g. bespoke WDM models) that the current hmf/HMFcalc fitting-function set does not cover.

## Inputs

- Cosmological parameter set (default PLANCK15: Omega_c, Omega_Lambda, sigma_8, spectral index n)
- Transfer function T(k) computed by CAMB or supplied as a CAMB-format file
- Mass grid specification (default 10^10-10^15 Msun/h) and redshift z
- Choice of f(sigma) fitting function (e.g. Tinker 2008) and window function (top-hat)

## Outputs

- Differential halo mass function dndlnm (number of haloes per unit comoving volume per unit lnM)
- Cumulative halo mass function ngtm (n(>M))
- Mass grid M in Msun/h and mass variance sigma(M) at redshift z

## How to apply

Instantiate the hmf MassFunction class (default PLANCK15 cosmology, z=0, Tinker 2008 fitting function, 10^10-10^15 Msun/h). The transfer function is obtained from the public Code for Anisotropies in the Microwave Background (CAMB, Lewis et al. 2000) via pycamb, or read from a file that 'must be either produced by CAMB or in CAMB format'; it is interpolated with linear splines in log-space and re-gridded between k-bounds (optionally extrapolated to high k with Eisenstein & Hu's fitting formula via extrapolate_with_eh=True). Build the unnormalised power spectrum P0(k) = k^n * T^2, then normalise it to sigma_8 — the mass variance on the 8 h^-1 Mpc scale at z=0 evaluated with the top-hat window — before applying the linear growth factor D(z) to both P(k) and sigma(M,z). Evaluate the chosen f(sigma) fitting function and compute the differential HMF (rho_0/M * f(sigma) * K) and the cumulative HMF ngtm via linear spline interpolation and romberg integration; use the update() method for cascading, dependency-tracked parameter changes (cuts processing time by ~50% per parameter change and >94% when only redshift changes).

## Related tools

- **hmf** (Python engine that performs the sigma_8-normalised power-spectrum calculation, top-hat mass-variance integration, and HMF evaluation (dndlnm, ngtm)) — https://github.com/halomod/hmf
- **CAMB** (Computes the transfer functions required for the linear power spectrum underlying the HMF calculation) — http://camb.info
- **pycamb** (Python bindings that let hmf call CAMB's transfer-function routines (pycamb.transfers returns k, T_i(k,z), sigma8(z))) — https://github.com/joezuntz/pycamb
- **HMFcalc** (Web frontend (http://hmf.icrar.org) to the hmf engine for interactive HMF exploration)

## Examples

```
from hmf import MassFunction; mf = MassFunction(); m, dndlnm = mf.m, mf.dndlnm
```

## Evaluation signals

- The mass grid spans the full default 10^10-10^15 Msun/h range and dndlnm values fall in the expected range for the default PLANCK15 cosmology at z=0.
- The computed dn/dlnM is consistent with the independent codes hmf was tested against — genmf, MF_Code and the yt-project HMF calculator.
- The power spectrum is normalised to sigma_8 (mass variance on the 8 h^-1 Mpc scale at z=0) with the top-hat window function, so recomputing sigma on 8 h^-1 Mpc returns the input sigma_8.
- The cumulative HMF ngtm obtained by linear spline interpolation and romberg integration matches the integral of dndlnm over mass.
- Supplying a transfer-function file not produced by CAMB (or not in CAMB format) raises an error rather than silently succeeding.

## Limitations

- The pycamb bindings are outdated: the README states 'This code is out of data and no longer works with recent versions of camb. Since camb itself now ships with python bindings you should use those instead.'
- Transfer functions must come from CAMB or a CAMB-format file; extrapolation to high k relies on the Eisenstein & Hu fitting formula (extrapolate_with_eh=True) rather than the numerical transfer function itself.
- The standard CDM HMF rises approximately as M^-1.8 at low mass; viable WDM models predict fewer haloes, so the top-hat mass-variance result is model-dependent and WDM requires the extra T_WDM(k) power-spectrum correction.
- Any CAMB 'STOP' statement (e.g. setting lmax too low) causes the whole Python process to exit when driving CAMB through pycamb.

## Evidence

- [methods] sigma_8 normalisation of the power spectrum: [evidence span withheld — anchor status: not_found]
