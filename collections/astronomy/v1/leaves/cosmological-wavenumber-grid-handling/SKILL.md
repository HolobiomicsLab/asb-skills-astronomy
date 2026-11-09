---
name: cosmological-wavenumber-grid-handling
description: Reference for ingesting a CAMB-format transfer function T(k), interpolating
  it in log-space with linear splines, and re-gridding it onto hmf's k-bounds before
  power-spectrum or HMF computation.
when_to_use_negative:
- The transfer-function file is not produced by CAMB and is not in CAMB format (hmf
  cannot read it directly).
- You need to recompute the transfer function itself from new cosmological parameters
  rather than interpolate/re-grid an existing one (that is CAMB's job, not this grid-handling
  step).
tools:
- name: hmf
  role: Performs log-space linear-spline interpolation of the transfer function and
    re-grids it between k-bounds before power-spectrum and HMF computation
  repo: https://github.com/steven-murray/HMF
- name: CAMB
  role: Computes the linear transfer function T(k) that is ingested onto the k-grid
- name: pycamb
  role: Python bindings exposing CAMB transfer functions (returns k, T_i(k,z), sigma8)
    for loading onto the grid
  repo: https://github.com/joezuntz/pycamb
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2013.11.001
    title: 'HMFcalc: An online tool for calculating dark matter halo mass functions'
  source_content_hash: sha256:c4c970374bef1cfdedf69b20e40f3f59530695f5158bbc19cdb236d0d6527acc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/steven-murray/HMF
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
yes_rate: 0.625
n_claims: 8
---

# cosmological-wavenumber-grid-handling

## Summary

Handles the shared wavenumber (k) grid on which CAMB-produced transfer functions are ingested, log-space interpolated and re-gridded inside the hmf/HMFcalc pipeline, so that downstream quantities (P(k), sigma(M), the HMF) are evaluated on a consistent k-basis across models such as CDM and WDM.

## When to use

Reference for ingesting a CAMB-format transfer function T(k), interpolating it in log-space with linear splines, and re-gridding it onto hmf's k-bounds before power-spectrum or HMF computation.

## When NOT to use

- The transfer-function file is not produced by CAMB and is not in CAMB format (hmf cannot read it directly).
- You need to recompute the transfer function itself from new cosmological parameters rather than interpolate/re-grid an existing one (that is CAMB's job, not this grid-handling step).

## Inputs

- Linear matter transfer function T(k) from CAMB (via pycamb) or a file in CAMB format
- k-grid bounds (default hmf range ~1e-8 to ~2000 h/Mpc)
- Choice of extrapolation flag (extrapolate_with_eh) for extending T(k) beyond the CAMB k-range

## Outputs

- T(k) re-gridded onto a shared log-space k-grid suitable for P0(k) = k^n T^2 evaluation
- Consistent k-basis for downstream sigma(M), growth-factor application and HMF computation

## How to apply

Obtain the linear transfer function either directly from CAMB (via pycamb) or by loading a file, which must be 'produced by CAMB or in CAMB format'. Interpolate T(k) with linear splines applied to the function in log-space, then re-grid between the package's k-bounds (hmf's default range runs from kmin ~ 1e-8 to kmax ~ 2000 h/Mpc). If the CAMB output does not reach the required high-k limit, enable extrapolate_with_eh=True so that Eisenstein & Hu's fitting formula extends the transfer function to higher k. Evaluate every model variant (e.g. CDM vs WDM with the Bode et al. T_X(k) filter) on this same shared grid so spectra can be tabulated column-by-column and compared like-for-like. The rationale is that all downstream steps — P0(k) = k^n T^2, sigma_8 normalisation, growth-factor application — depend on the transfer function, so grid consistency controls their validity.

## Related tools

- **hmf** (Performs log-space linear-spline interpolation of the transfer function and re-grids it between k-bounds before power-spectrum and HMF computation) — https://github.com/steven-murray/HMF
- **CAMB** (Computes the linear transfer function T(k) that is ingested onto the k-grid)
- **pycamb** (Python bindings exposing CAMB transfer functions (returns k, T_i(k,z), sigma8) for loading onto the grid) — https://github.com/joezuntz/pycamb

## Examples

```
import pycamb; k, T, s8 = pycamb.transfers(H0=72, omegac=0.36, omegav=0.6)
```

## Evaluation signals

- T(k) interpolation is exact at the original CAMB sample points (no drift when re-evaluated on the source grid).
- Re-gridded T(k) is smooth and monotone-decreasing in log-space across the full k-range, with no spline artefacts between k-bounds.
- If extrapolate_with_eh=True, the Eisenstein & Hu extension agrees with CAMB's T(k) in the overlap region before diverging only at k above CAMB's kmax.
- All model variants (e.g. CDM vs WDM m_X = 0.5, 1.0, 2.0 keV) are evaluated on an identical k-grid, so tabulated columns share row values.
- Downstream P0(k) = k^n T^2 computed from the re-gridded T(k) reproduces the sigma_8-normalised spectrum within tolerance.

## Limitations

- Input transfer functions must be produced by CAMB or be in CAMB format; other file layouts are rejected.
- Linear-spline interpolation in log-space can introduce small errors where T(k) has strong curvature; sample density from CAMB matters.
- If CAMB's k-range is insufficient, extrapolation relies on the Eisenstein & Hu fitting formula, which is a CDM-based approximation and may not represent modified (e.g. WDM) transfer behaviour at high k.

## Evidence

- [readme] pycamb transfers API returns k and T_i(k,z): [evidence span withheld — anchor status: not_found]
