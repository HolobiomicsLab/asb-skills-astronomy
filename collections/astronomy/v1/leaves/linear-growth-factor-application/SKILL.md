---
name: linear-growth-factor-application
description: 'Use when computing the halo mass function at redshift z != 0 (or normalising
  the power spectrum at z=0 via sigma8): the z-independent transfer function output
  must be grown to redshift z before evaluating f(sigma) and the HMF.'
when_to_use_negative:
- When the requested output is the z=0 HMF only — the growth factor d(0)=1 is trivial
  and the workflow applies the default growth already
- When the power spectrum already comes from a file/CAMB run evaluated at the target
  redshift, so applying d^2(z) again would double-count the growth
- When modelling non-linear growth or scale-dependent growth modifications beyond
  hmf's linear ODE solver
edam_operation: http://edamontology.org/operation_2426
tools:
- name: hmf
  role: Python library whose MassFunction class applies the linear growth factor to
    P(k) and sigma(M) inside the cascading update() pipeline
  repo: https://github.com/halomod/hmf
- name: CAMB
  role: Supplies the z=0 transfer function from which P0(k) is built before growth
    scaling
  repo: https://github.com/joezuntz/pycamb
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2013.11.001
    title: 'HMFcalc: An online tool for calculating dark matter halo mass functions'
  source_content_hash: sha256:76c842054f736a403471a1281689042b00dea950fc4734d489d1333960a77ba8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/halomod/hmf
  tools:
  - CAMB
  - Python
  - hmf
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# linear-growth-factor-application

## Summary

Rescales the unnormalised matter power spectrum P0(k) and mass variance sigma_0(M) by the linear growth factor d(z) to obtain redshift-dependent quantities in the hmf/HMFcalc halo mass function pipeline.

## When to use

Use when computing the halo mass function at redshift z != 0 (or normalising the power spectrum at z=0 via sigma8): the z-independent transfer function output must be grown to redshift z before evaluating f(sigma) and the HMF.

## When NOT to use

- When the requested output is the z=0 HMF only — the growth factor d(0)=1 is trivial and the workflow applies the default growth already
- When the power spectrum already comes from a file/CAMB run evaluated at the target redshift, so applying d^2(z) again would double-count the growth
- When modelling non-linear growth or scale-dependent growth modifications beyond hmf's linear ODE solver

## Inputs

- Unnormalised power spectrum P0(k) = k^n T^2 from a CAMB transfer function
- Sigma8-normalised P(k) and mass variance sigma_0(M) on the default log mass grid (0.05 dex bins)
- Target redshift z passed as a keyword argument

## Outputs

- Redshift-scaled power spectrum P(k,z) = P0(k)*d^2(z)
- Redshift-scaled mass variance sigma(M,z) = sigma_0(M)*d(z)
- Redshift-dependent dndlnm and ngtm HMF curves

## How to apply

In hmf, the transfer function from CAMB is first converted to an unnormalised power spectrum P0(k) = k^n T^2 and normalised by sigma8. The linear growth factor d(z) is then applied to both the power spectrum and the mass variance — P(k) <- P0(k)*d^2(z) and sigma(M,z) <- sigma_0(M)*d(z) — because the transfer function is computed at z=0 while the HMF fitting function f(sigma) depends on sigma(M,z) evaluated at the target redshift. Internally hmf solves the growth factor ODE as a coupled first-order system with initial conditions set in the radiation-dominated era, where D(a) ~ 2*a_eq/3 and D'(a) ~ 0 as a -> 0. Only after this growth scaling should dndlnm and ngtm be computed; updating z via the cascading update() method correctly invalidates and recomputes the growth-dependent quantities.

## Related tools

- **hmf** (Python library whose MassFunction class applies the linear growth factor to P(k) and sigma(M) inside the cascading update() pipeline) — https://github.com/halomod/hmf
- **CAMB** (Supplies the z=0 transfer function from which P0(k) is built before growth scaling) — https://github.com/joezuntz/pycamb

## Examples

```
from hmf import MassFunction; mf = MassFunction(z=0.0); mf.update(z=1.0)  # recomputes d(z), P(k,z), sigma(M,z), dndlnm
```

## Evaluation signals

- At z=0 the growth factor d(z)=1, so the scaled P(k) and sigma(M) must exactly match the unnormalised sigma8-normalised quantities
- Growth factor initial conditions must satisfy D(a) ~ 2*a_eq/3 and D'(a) ~ 0 as a -> 0 in the radiation-dominated era
- Updating only z via update() must leave the transfer function untouched and recompute only growth-dependent downstream quantities (P(k), sigma, f(sigma), dndlnm, ngtm)
- HMF amplitude should decrease at high redshift for a fixed mass, consistent with d^2(z) scaling of the power spectrum
- Normalisation must remain fixed by sigma8 at z=0; changing z must not alter the sigma8 normalisation itself

## Limitations

- The growth factor is linear-theory only; hmf's implementation was subject to a bug fixed in halomod/hmf PR #270, so older versions may produce incorrect growth scaling
- The ODE solver assumes standard LCDM-like growth; non-standard dark energy or dark matter models may not be supported by the built-in growth integration
- Growth must be applied to both P(k) and sigma(M) consistently; applying it to only one breaks the HMF normalisation

## Evidence
