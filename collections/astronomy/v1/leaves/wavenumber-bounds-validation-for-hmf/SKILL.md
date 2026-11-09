---
name: wavenumber-bounds-validation-for-hmf
description: Use when computing or reproducing the mass-variance σ(M) integrals in
  hmf and you need to confirm the k-limits (default ~1e-8 to 2000 h/Mpc) are wide
  enough that the top-hat window integrals are converged for the halo masses scanned.
when_to_use_negative:
- The analysis does not involve the σ(M) mass-variance integral or top-hat window
  function (e.g. only the differential HMF fitting functions are being evaluated)
  — the kR rule is specific to window-function integral convergence.
- A custom, externally validated k-grid is already supplied and its adequacy has been
  established by a different convergence test.
- Using WDM-corrected or sharply truncated power spectra where the 95% rule's assumptions
  about integral containment may not transfer.
tools:
- name: hmf
  role: Provides the default kR bound-check with warning behaviour and the linear-spline
    log-space interpolation and Romberg integration scheme being reproduced
  repo: https://github.com/halomod/hmf
- name: CAMB / pycamb
  role: Generates the fiducial transfer function from which P0(k) = k^n * T^2 is constructed
  repo: https://github.com/joezuntz/pycamb
- name: Python
  role: Implementation language for re-gridding the k-grid (4097 steps) and running
    the Romberg convergence scan
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2013.11.001
    title: 'HMFcalc: An online tool for calculating dark matter halo mass functions'
  source_content_hash: sha256:3dc1a074c070215a1d2d3d2eb0c1be521d27b2883b6bbe07c400f304c2de5420
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/halomod/hmf
  tools:
  - CAMB / pycamb
  - Python
  - hmf
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# wavenumber-bounds-validation-for-hmf

## Summary

Validates that the k-range used for the σ(M) mass-variance integrals satisfies the kR convergence bounds derived from the 95% integral-fraction rule (k_min*R_max < 0.1 and k_max*R_min > 3), mirroring the warning logic hmf applies by default.

## When to use

Use when computing or reproducing the mass-variance σ(M) integrals in hmf and you need to confirm the k-limits (default ~1e-8 to 2000 h/Mpc) are wide enough that the top-hat window integrals are converged for the halo masses scanned.

## When NOT to use

- The analysis does not involve the σ(M) mass-variance integral or top-hat window function (e.g. only the differential HMF fitting functions are being evaluated) — the kR rule is specific to window-function integral convergence.
- A custom, externally validated k-grid is already supplied and its adequacy has been established by a different convergence test.
- Using WDM-corrected or sharply truncated power spectra where the 95% rule's assumptions about integral containment may not transfer.

## Inputs

- CAMB-format transfer function T(k) (file or via pycamb) or hmf's pre-calculated transfer function from github.com/steven-murray/HMF
- Spectral index n and k-grid bounds (default ~1e-8 to 2000 h/Mpc) with 4097 re-gridding steps
- List of halo masses / radii R to scan

## Outputs

- kR cutoff values at which partial integrals retain 95% of the total, per halo mass R
- Integral-fraction table and pass/fail flags for the inequalities k_min*R_max < 0.1 and k_max*R_min > 3
- kR_convergence_results.csv (cutoffs, fractions, bound-check flags)

## How to apply

Build the unnormalised power spectrum P0(k) = k^n * T^2 from a CAMB transfer function, interpolating with linear splines in log-space and re-gridding between k-bounds in 4097 steps as hmf does. Define the top-hat window W(x) = 3(sin x - x cos x)/x^3 and integrate the mass-variance integrand over [0, kR] and [kR, inf) with Romberg integration using 4097 steps. For each halo mass R, locate the kR cutoffs at which the partial integrals retain 95% of the total integral, then check the bound conditions k_min*R_max < 0.1 and k_max*R_min > 3. If either inequality fails, flag a violation (hmf itself issues a warning when the bounds are not obeyed).

## Related tools

- **hmf** (Provides the default kR bound-check with warning behaviour and the linear-spline log-space interpolation and Romberg integration scheme being reproduced) — https://github.com/halomod/hmf
- **CAMB / pycamb** (Generates the fiducial transfer function from which P0(k) = k^n * T^2 is constructed) — https://github.com/joezuntz/pycamb
- **Python** (Implementation language for re-gridding the k-grid (4097 steps) and running the Romberg convergence scan)

## Examples

```
>>> from hmf import get_base_components
```

## Evaluation signals

- The 95% integral-fraction criterion is met for all scanned halo masses R.
- Both inequalities hold for the default k-limits (~1e-8 to 2000 h/Mpc): k_min*R_max < 0.1 and k_max*R_min > 3.
- The re-gridded P0(k) matches hmf's construction (linear splines in log-space, 4097 steps).
- Any deliberately narrowed k-range triggers a violation flag, matching hmf's warning behaviour in tools.py.
- kR_convergence_results.csv contains cutoffs, fractions, and pass/fail flags for every scanned mass.

## Limitations

- The 95% rule was derived for the top-hat window function; other filters are not covered by these bounds.
- hmf's extrapolate_with_eh option (Eisenstein & Hu fitting formula) may extend transfer functions to higher k, altering effective bounds.
- Bounds depend on the scanned R range: very large R_max pushes k_min*R_max above 0.1 for the default k_min.

## Evidence
