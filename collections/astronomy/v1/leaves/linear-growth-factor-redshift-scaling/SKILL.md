---
name: linear-growth-factor-redshift-scaling
description: Use when computing a halo mass function (HMF) or mass variance at nonzero
  redshift from a z=0 transfer function, or when updating an existing hmf calculation
  for a changed redshift without recomputing the transfer function.
when_to_use_negative:
- When a cosmological parameter (not just redshift) changes — the transfer function
  must be recomputed via CAMB and growth rescaling alone is invalid.
- When nonlinear growth or WDM suppression is required — the pipeline's growth application
  is linear (P <- P0 * D^2) and the WDM correction is applied separately as P <- P
  * T_WDM^2.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: hmf
  role: Engine that solves the growth ODE, applies D(z) to P(k) and sigma(M), and
    performs cascading redshift-only updates via update()
  repo: https://github.com/halomod/hmf
- name: CAMB
  role: Computes the z=0 transfer functions used as the base input for the power spectrum
    that is then growth-rescaled
  repo: https://github.com/joezuntz/pycamb
- name: Python
  role: Implementation language of hmf and its growth-factor ODE solver
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2013.11.001
    title: 'HMFcalc: An online tool for calculating dark matter halo mass functions'
  source_content_hash: sha256:79a0eb6c6b642dd800d5424f8f3c236acffcaf1acd98a7c36fd2d2fede877550
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/halomod/hmf
  tools:
  - CAMB
  - Python
  - hmf
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# linear-growth-factor-redshift-scaling

## Summary

Applies the linear growth factor D(z) to rescale the z=0 power spectrum and mass variance to a target redshift in the hmf/HMFcalc pipeline, solving the growth ODE as a coupled first-order system with radiation-era initial conditions, so that redshift changes require only rescaling rather than full recomputation.

## When to use

Use when computing a halo mass function (HMF) or mass variance at nonzero redshift from a z=0 transfer function, or when updating an existing hmf calculation for a changed redshift without recomputing the transfer function.

## When NOT to use

- When a cosmological parameter (not just redshift) changes — the transfer function must be recomputed via CAMB and growth rescaling alone is invalid.
- When nonlinear growth or WDM suppression is required — the pipeline's growth application is linear (P <- P0 * D^2) and the WDM correction is applied separately as P <- P * T_WDM^2.

## Inputs

- Transfer function T(k) computed by CAMB (or a CAMB-format file)
- Unnormalised power spectrum P0(k) = k^n T^2 normalised by sigma_8
- Target redshift z (scalar)

## Outputs

- Redshift-scaled power spectrum P(k) = P0(k) D(z)^2
- Redshift-scaled mass variance sigma(M, z) = sigma0(M) D(z)
- Downstream HMF quantities (dndlnm, ngtm) at redshift z

## How to apply

In hmf, obtain the unnormalised power spectrum P0(k) = k^n T^2 from the CAMB transfer function, normalise with sigma_8, then apply the linear growth factor: P(k) <- P0(k) D(z)^2 and sigma(M,z) <- sigma0(M) D(z). D(a) is obtained by solving the growth ODE as a coupled first-order system with initial conditions set in the radiation-dominated era, where D(a) ~ 2 a_eq/3 and D'(a) ~ 0 as a -> 0. When only redshift changes, call update(z=...) so the cascading dependency-tracking mechanism rescales growth-dependent quantities (power, sigma, dndlnm, ngtm) instead of recomputing the CAMB transfer function; the paper reports this reduces processing time by more than 94% versus full recomputation.

## Related tools

- **hmf** (Engine that solves the growth ODE, applies D(z) to P(k) and sigma(M), and performs cascading redshift-only updates via update()) — https://github.com/halomod/hmf
- **CAMB** (Computes the z=0 transfer functions used as the base input for the power spectrum that is then growth-rescaled) — https://github.com/joezuntz/pycamb
- **Python** (Implementation language of hmf and its growth-factor ODE solver)

## Examples

```
from hmf import MassFunction; mf = MassFunction(zmax=0); mf.update(z=1.0)  # rescales power/sigma via D(z)
```

## Evaluation signals

- P(k, z) must equal P0(k) * D(z)^2 and sigma(M, z) must equal sigma0(M) * D(z) exactly when re-derived from the stored z=0 quantities.
- The growth solution should satisfy D(a) ~ 2 a_eq/3 and D'(a) ~ 0 as a -> 0 (radiation-era initial conditions).
- A redshift-only update() call should trigger recomputation of growth-dependent quantities only (power, sigma, dndlnm, ngtm), not the CAMB transfer function.
- Benchmark: redshift-only cascading update should reduce processing time by more than 94% versus full recomputation.
- HMF at z should agree with a from-scratch recomputation at the same z within interpolation/integration tolerance (Romberg integration, 4097-step log-space regridding).

## Limitations

- The growth application is linear only (P <- P0 D^2); it does not capture nonlinear growth corrections.
- A historical bug in the growth factor implementation existed in hmf and was fixed in https://github.com/halomod/hmf/pull/270, so verify the installed version.
- The >94% time saving applies only to redshift-only changes; changing other parameters still triggers transfer-function recomputation.

## Evidence
