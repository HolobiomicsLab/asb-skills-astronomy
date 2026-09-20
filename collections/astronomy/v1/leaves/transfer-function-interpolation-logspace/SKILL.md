---
name: transfer-function-interpolation-logspace
description: Reference for reproducing how hmf interpolates and re-grids CAMB transfer
  functions (CAMB-format files or pycamb output) in log-space with linear splines
  before power-spectrum construction.
when_to_use_negative:
- You need a transfer function at k values beyond the tabulated range and cannot accept
  an Eisenstein & Hu extrapolation (extrapolate_with_eh=True) instead of an actual
  CAMB evaluation
- Your transfer function is not produced by CAMB and is not in CAMB format — hmf will
  not read it and the interpolation step cannot proceed
- You are re-running CAMB itself to evaluate T(k) directly at each required k rather
  than re-gridding a fixed table
tools:
- name: hmf
  role: Implements the log-space linear-spline interpolation and re-gridding of the
    transfer function between k-bounds as part of its HMF calculation
  repo: https://github.com/halomod/hmf
- name: CAMB
  role: Public code used to compute the transfer functions that are then interpolated;
    accessed through pycamb
- name: pycamb
  role: Python bindings whose transfers routine returns k, T_i(k,z) and sigma8(z),
    supplying the transfer-function input for interpolation
  repo: https://github.com/joezuntz/pycamb
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2013.11.001
    title: 'HMFcalc: An online tool for calculating dark matter halo mass functions'
  source_content_hash: sha256:0555ae89bcd823755f7500841f2943bf9f4aef8a6ecb285cc2a101554bfa08fc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/halomod/hmf
  tools:
  - CAMB
  - Python
  - hmf
  - pycamb
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# transfer-function-interpolation-logspace

## Summary

Interpolate a CAMB-produced matter transfer function T(k) with linear splines in log-space and re-grid it between k-bounds, so that hmf can form the unnormalised power spectrum P0(k) = k^n * T^2. This is the standard pre-processing step in the hmf HMF pipeline between transfer-function acquisition and sigma_8 normalisation.

## When to use

Reference for reproducing how hmf interpolates and re-grids CAMB transfer functions (CAMB-format files or pycamb output) in log-space with linear splines before power-spectrum construction.

## When NOT to use

- You need a transfer function at k values beyond the tabulated range and cannot accept an Eisenstein & Hu extrapolation (extrapolate_with_eh=True) instead of an actual CAMB evaluation
- Your transfer function is not produced by CAMB and is not in CAMB format — hmf will not read it and the interpolation step cannot proceed
- You are re-running CAMB itself to evaluate T(k) directly at each required k rather than re-gridding a fixed table

## Inputs

- Transfer function T(k) computed by CAMB via pycamb (k, T_i(k,z), sigma8(z) arrays)
- Transfer-function file in CAMB format
- Cosmological parameters (e.g. PLANCK15 defaults) and k-bounds for re-gridding

## Outputs

- Re-gridded transfer function T(k) on the target k-grid
- Unnormalised power spectrum P0(k) = k^n * T^2 ready for sigma_8 normalisation

## How to apply

First obtain the transfer function either by calling CAMB (via pycamb's transfers routine, which returns k, T_i(k,z) and sigma8(z)) or by reading a file; the file must be produced by CAMB or be in CAMB format. Interpolate the transfer function with linear splines applied on the function in log-space, then re-grid the interpolated values onto the desired k-grid between the specified k-bounds. The re-gridded T(k) feeds directly into the unnormalised power spectrum P0(k) = k^n * T^2, which is subsequently normalised via sigma_8. For input files whose tabulated k-range does not cover the required range, set extrapolate_with_eh=True so Eisenstein & Hu's fitting formula extrapolates the transfer function to higher k values. The rationale is that spline interpolation in log-space reproduces the steep, near-power-law falloff of T(k) accurately enough that the re-gridded HMF matches reference codes (genmf, MF_Code, yt-project HMF calculator).

## Related tools

- **hmf** (Implements the log-space linear-spline interpolation and re-gridding of the transfer function between k-bounds as part of its HMF calculation) — https://github.com/halomod/hmf
- **CAMB** (Public code used to compute the transfer functions that are then interpolated; accessed through pycamb)
- **pycamb** (Python bindings whose transfers routine returns k, T_i(k,z) and sigma8(z), supplying the transfer-function input for interpolation) — https://github.com/joezuntz/pycamb

## Examples

```
from hmf import MassFunction; mf = MassFunction(transfer_filename='my_camb_transfer.dat'); T = mf.transfer; dndlnm = mf.dndlnm
```

## Evaluation signals

- The interpolated transfer function reproduces the tabulated CAMB values at the original k-points (spline interpolation is exact at nodes)
- Downstream dn/dlnM values computed from the re-gridded T(k) agree with reference implementations (genmf, MF_Code, yt-project HMF calculator) with excellent consistency
- The re-gridded table spans the full configured k-bounds with no gaps or out-of-range k values
- The unnormalised power spectrum satisfies P0(k) = k^n * T^2 and, after sigma_8 normalisation, the mass variance on the 8 h^-1 Mpc scale equals the input sigma_8 at z=0
- If a transfer-function file is supplied, it parses successfully as CAMB-format output

## Limitations

- The legacy joezuntz/pycamb bindings are out of date and no longer work with recent versions of CAMB, which now ships its own Python bindings
- Interpolation uses linear splines on log-space; accuracy degrades outside the tabulated k-range unless extrapolate_with_eh=True is used, which substitutes an Eisenstein & Hu fitting-formula extrapolation rather than a true CAMB evaluation
- The pipeline's power-spectrum machinery assumes the WDM correction and growth factor are applied after interpolation, so replacing this step changes those downstream quantities as well

## Evidence

- [readme] pycamb supplies the transfer-function arrays used for interpolation: [evidence span withheld — anchor status: not_found]
