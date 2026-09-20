---
name: power-spectrum-construction-from-transfer-function
description: Use when building the primordial/unnormalised power spectrum from a transfer
  function as the first step of a halo mass function (HMF) or mass-variance calculation,
  and when the k-grid must be checked for convergence (k_min*R_max < 0.1, k_max*R_min
  > 3).
when_to_use_negative:
- You already have a measured/tabulated P(k) from a simulation or observation — this
  skill only builds P0(k) from a transfer function, it does not fit or smooth an existing
  P(k).
- Your analysis targets non-linear power or the full halo model beyond the linear
  regime that hmf's sigma(M) pipeline assumes.
- The transfer function comes from a code other than CAMB and cannot be cast into
  CAMB format — hmf requires 'either produced by CAMB or in CAMB format'.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3293
tools:
- name: hmf
  role: Engine that performs the spline interpolation, re-gridding, normalisation,
    and k-range convergence checking on the transfer function and power spectrum
  repo: https://github.com/steven-murray/HMF
- name: CAMB
  role: Computes the fiducial matter transfer function T(k) that is squared to form
    P0(k)
  repo: https://github.com/joezuntz/pycamb
- name: pycamb
  role: Python bindings exposing pycamb.transfers() to obtain k, T_i(k,z) from CAMB
  repo: https://github.com/joezuntz/pycamb
- name: HMFcalc
  role: Web frontend exposing the same power-spectrum construction pipeline for interactive
    use
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2013.11.001
    title: 'HMFcalc: An online tool for calculating dark matter halo mass functions'
  source_content_hash: sha256:624686fe17cd497ced3c3e1265ca68dfdaf95d5883767bc4a9972ee3927d7bb2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/steven-murray/HMF
  tools:
  - CAMB
  - HMFcalc
  - Python
  - hmf
  - pycamb
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3293
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# power-spectrum-construction-from-transfer-function

## Summary

Constructs the unnormalised linear power spectrum P0(k) = k^n * T^2 by squaring a CAMB-generated transfer function, then re-grids it in log-space with linear splines (4097 steps) exactly as hmf does, so that downstream σ(M) integrals inherit a converged k-range.

## When to use

Use when building the primordial/unnormalised power spectrum from a transfer function as the first step of a halo mass function (HMF) or mass-variance calculation, and when the k-grid must be checked for convergence (k_min*R_max < 0.1, k_max*R_min > 3).

## When NOT to use

- You already have a measured/tabulated P(k) from a simulation or observation — this skill only builds P0(k) from a transfer function, it does not fit or smooth an existing P(k).
- Your analysis targets non-linear power or the full halo model beyond the linear regime that hmf's sigma(M) pipeline assumes.
- The transfer function comes from a code other than CAMB and cannot be cast into CAMB format — hmf requires 'either produced by CAMB or in CAMB format'.

## Inputs

- Transfer function T(k) produced by CAMB (via pycamb bindings) or read from a file in CAMB format
- Scalar spectral index n (for P0(k) = k^n * T^2)
- sigma_8 for power-spectrum normalisation
- Optional: WDM model selection triggering the T_WDM(k) suppression factor
- Redshift z for the linear growth factor D(z)

## Outputs

- Interpolated, re-gridded power spectrum P(k) on 4097 log-space k-steps between the k-bounds (~1e-8 to 2000 h/Mpc by default)
- k-range convergence status against the k_min*R_max < 0.1 and k_max*R_min > 3 top-hat window criteria (warning if violated)
- Normalised P(k) and sigma_0(M) suitable for downstream HMF fitting-function evaluation

## How to apply

Obtain a fiducial transfer function T(k) via CAMB (pycamb bindings) or import a file in CAMB format; compute the unnormalised power spectrum P0(k) = k^n * T^2 where n is the spectral index. Interpolate the transfer function with linear splines on the function in log-space and re-grid between the k-bounds in 4097 steps, mirroring hmf's internal scheme. Normalise using sigma_8 (P(k) = (sigma_8/sigma_8_hat)^2 * P0(k)) and, at redshift z > 0, apply the linear growth factor D(z) to both P(k) and sigma_0(M). Validate the resulting k-limits (hmf defaults ~1e-8 to 2000 h/Mpc) against the top-hat window-function convergence rule k_min*R_max < 0.1 and k_max*R_min > 3, which hmf's tools.py enforces by warning; if T(k) needs extension beyond the CAMB grid, hmf offers extrapolate_with_eh=True using Eisenstein & Hu's fitting formula.

## Related tools

- **hmf** (Engine that performs the spline interpolation, re-gridding, normalisation, and k-range convergence checking on the transfer function and power spectrum) — https://github.com/steven-murray/HMF
- **CAMB** (Computes the fiducial matter transfer function T(k) that is squared to form P0(k)) — https://github.com/joezuntz/pycamb
- **pycamb** (Python bindings exposing pycamb.transfers() to obtain k, T_i(k,z) from CAMB) — https://github.com/joezuntz/pycamb
- **HMFcalc** (Web frontend exposing the same power-spectrum construction pipeline for interactive use)

## Examples

```
from pycamb import transfers; k, T, sigma8 = transfers(2000, H0=70., omegab=0.04, omegav=0.6, omegac=0.36); P0 = k**1.0 * T**2
```

## Evaluation signals

- P0(k) equals k^n * T^2 pointwise on the re-gridded log-space k-array (4097 steps) before normalisation.
- After sigma_8 normalisation, sigma(sigma_8 mass) reproduces the input sigma_8 within numerical tolerance of the Romberg integration used by hmf.
- The k-bounds satisfy k_min*R_max < 0.1 and k_max*R_min > 3 for all scanned halo radii R; hmf issues a warning on violation.
- Transfer-function interpolation residuals are consistent with linear-spline accuracy in log-space; extrapolation beyond the grid only via extrapolate_with_eh=True (Eisenstein & Hu).
- Growth-factor application maps sigma_0(M) to sigma(M,z) with D(z) consistent with the ODE solution initialised at D ≈ 2*a_eq/3 in the radiation-dominated era.

## Limitations

- pycamb is outdated: 'This code is out of data and no longer works with recent versions of camb. Since camb itself now ships with python bindings you should use those instead.'
- A user-supplied transfer-function file must be produced by CAMB or in CAMB format; other formats are rejected.
- Extrapolation beyond the CAMB k-grid relies on the Eisenstein & Hu fitting formula (extrapolate_with_eh=True), not on the CAMB spline itself.
- Any CAMB error that triggers a Fortran STOP propagates and terminates the Python process (noted in the pycamb README).

## Evidence

- [readme] The pycamb README warns that the package is deprecated in favour of CAMB's native Python bindings: [evidence span withheld — anchor status: not_found]
