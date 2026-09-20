---
name: power-spectrum-normalisation-sigma8
description: Use when computing an HMF, mass variance sigma(M), or differential mass
  function dndlnm from a linear power spectrum and the amplitude must be fixed to
  the observed sigma_8 (e.g. default hmf cosmology) rather than left unnormalised.
when_to_use_negative:
- The power spectrum is already normalised or supplied as a measured P(k) with its
  own amplitude — renormalising would double-count the amplitude constraint
- The analysis needs absolute halo abundances from simulation-calibrated fits rather
  than a sigma_8-constrained analytic HMF, or the transfer function comes from a source
  whose normalisation convention differs from CAMB's sigma_8 output
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: hmf
  role: Python engine that performs the sigma_8 power-spectrum normalisation, growth-factor
    application, fitting-function evaluation and cascading update() recalculation
    of dndlnm
  repo: https://github.com/steven-murray/HMF
- name: CAMB
  role: Computes the transfer functions T(k) that feed the unnormalised spectrum P0(k)
    = k^n T^2 prior to sigma_8 normalisation
  repo: https://github.com/joezuntz/pycamb
- name: HMFcalc
  role: Web frontend exposing the hmf engine's sigma_8-normalised HMF calculations
    with configurable cosmological parameters
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2013.11.001
    title: 'HMFcalc: An online tool for calculating dark matter halo mass functions'
  source_content_hash: sha256:5d1c16f026d42f38327f23df1ba58a9d89bdb9257fff6a124680cb5c2cb11f5f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/steven-murray/HMF
  tools:
  - CAMB
  - HMFcalc
  - Python
  - hmf
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# power-spectrum-normalisation-sigma8

## Summary

Normalise the linear matter power spectrum to sigma_8 — the rms mass variance on the 8 h^-1 Mpc scale at z=0 computed with a top-hat window — before feeding it into halo mass function (HMF) fitting-function calculations, as implemented in the hmf engine behind HMFcalc.

## When to use

Use when computing an HMF, mass variance sigma(M), or differential mass function dndlnm from a linear power spectrum and the amplitude must be fixed to the observed sigma_8 (e.g. default hmf cosmology) rather than left unnormalised.

## When NOT to use

- The power spectrum is already normalised or supplied as a measured P(k) with its own amplitude — renormalising would double-count the amplitude constraint
- The analysis needs absolute halo abundances from simulation-calibrated fits rather than a sigma_8-constrained analytic HMF, or the transfer function comes from a source whose normalisation convention differs from CAMB's sigma_8 output

## Inputs

- Transfer function T(k) from CAMB (or a file in CAMB format)
- Scalar spectral index n and target sigma_8 value
- Top-hat window function and mass grid (default 10^10-10^15 Msun/h in hmf)
- Redshift z for applying the linear growth factor d(z)

## Outputs

- Normalised linear power spectrum P(k)
- Mass variance sigma(M, z)
- Differential mass function dndlnm and cumulative HMF ngtm

## How to apply

Start from the unnormalised power spectrum P0(k) = k^n T^2, where T(k) is the transfer function obtained from CAMB and interpolated with linear splines in log-space. Compute the mass variance sigma(M) by convolving P0(k) with the top-hat window function, then rescale the power spectrum by the factor (sigma_8_target / sigma_8_measured)^2 so the variance on the 8 h^-1 Mpc scale equals the specified sigma_8. At redshift z > 0, apply the linear growth factor via P(k) -> P0(k) d^2(z) and sigma(M,z) -> sigma_0(M) d(z) before evaluating the fitting function f(sigma) and the HMF (HMF = rho_0/M * f(sigma) * K). In the hmf Python package this chain is encapsulated in a MassFunction object whose default normalisation is sigma_8, so instantiating with defaults (PLANCK15 cosmology, Tinker 2008 fit, 10^10-10^15 Msun/h) already yields a correctly normalised dndlnm; changes to sigma_8 or other parameters propagate through the cascading update() method.

## Related tools

- **hmf** (Python engine that performs the sigma_8 power-spectrum normalisation, growth-factor application, fitting-function evaluation and cascading update() recalculation of dndlnm) — https://github.com/steven-murray/HMF
- **CAMB** (Computes the transfer functions T(k) that feed the unnormalised spectrum P0(k) = k^n T^2 prior to sigma_8 normalisation) — https://github.com/joezuntz/pycamb
- **HMFcalc** (Web frontend exposing the hmf engine's sigma_8-normalised HMF calculations with configurable cosmological parameters)

## Examples

```
from hmf import MassFunction; mf = MassFunction(sigma_8=0.8, z=0); dndlnm = mf.dndlnm
```

## Evaluation signals

- Verify the mass variance on the 8 h^-1 Mpc scale at z=0 equals the specified sigma_8 within numerical tolerance after normalisation
- Confirm the computed dndlnm covers the full default 10^10-10^15 Msun/h mass span and falls in the expected range for the default PLANCK15 cosmology (hmf was tested against genmf, MF_Code and the yt-project HMF calculator with excellent consistency)
- Check that at z>0 the growth factor was applied to both P(k) and sigma(M): sigma(M,z) = sigma_0(M) d(z), so derived HMFs scale consistently with d(z)
- After changing sigma_8 via update(), confirm only sigma-dependent downstream quantities (sigma, f(sigma), dndlnm, ngtm) are recalculated, consistent with the cascading dependency tracker
- For CDM vs WDM comparisons, confirm ratios dndlnm_WDM/dndlnm_CDM are order-unity at high mass with low-mass suppression increasing monotonically as m_X decreases

## Limitations

- Transfer-function interpolation uses linear splines in log-space between k-bounds, so accuracy degrades outside the CAMB k-range unless extrapolate_with_eh=True (Eisenstein & Hu fitting formula) is used
- The WDM branch multiplies P(k) by T_WDM^2 (Eq 14, Bode et al. 2001) only when a WDM model is explicitly selected; using CDM transfer functions with WDM particle masses silently omits the suppression
- pycamb as documented is dated (built for the January 2010 CAMB, Python 2.5-2.7); recent hmf workflows should use CAMB's own Python bindings
- Cumulative HMF ngtm relies on linear spline interpolation and extrapolation with romberg integration, so extreme mass-range extrapolation is less reliable

## Evidence

- [methods] Unnormalised spectrum built from transfer function: [evidence span withheld — anchor status: not_found]
- [other] hmf validates against independent codes: [evidence span withheld — anchor status: not_found]
