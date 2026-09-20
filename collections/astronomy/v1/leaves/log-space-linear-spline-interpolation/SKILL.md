---
name: log-space-linear-spline-interpolation
description: Use when computing the mass variance sigma(M), power spectrum P(k), or
  cumulative HMF integrals from a CAMB-format transfer function and you must match
  hmf's numerics (log-space linear splines, 4097-point re-grid, Romberg integration).
when_to_use_negative:
- You need a smooth analytic P(k) or window-function form rather than reproducing
  hmf's discrete re-grid — evaluate the analytic expression directly instead of splining.
- The transfer function is on an extremely coarse or irregular grid and you need accuracy
  beyond what linear splines in log-space provide (hmf offers extrapolate_with_eh=True
  using the Eisenstein & Hu fitting formula for high-k extrapolation as an alternative).
tools:
- name: hmf
  role: Reference implementation whose interpolation scheme (linear splines on log-space,
    4097-step re-grid) is being reproduced
  repo: https://github.com/steven-murray/HMF
- name: CAMB
  role: Generates the transfer functions that are interpolated in log-space
  repo: https://github.com/joezuntz/pycamb
- name: Python
  role: Language in which the spline interpolation and re-gridding are implemented
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2013.11.001
    title: 'HMFcalc: An online tool for calculating dark matter halo mass functions'
  source_content_hash: sha256:8dc3a22518064c94098ca1048ccd7a2a92af1bd323cbd8dc93ae15c5452af1a5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/steven-murray/HMF
  tools:
  - CAMB
  - Python
  - hmf
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# log-space-linear-spline-interpolation

## Summary

Reproduce hmf's interpolation strategy for the matter power spectrum: linear splines applied to the transfer function / unnormalised power spectrum P0(k)=k^n*T^2 in log-space, re-gridded over the k-bounds in 4097 steps, so that subsequent Romberg integrals for the mass variance sigma(M) are stable.

## When to use

Use when computing the mass variance sigma(M), power spectrum P(k), or cumulative HMF integrals from a CAMB-format transfer function and you must match hmf's numerics (log-space linear splines, 4097-point re-grid, Romberg integration).

## When NOT to use

- You need a smooth analytic P(k) or window-function form rather than reproducing hmf's discrete re-grid — evaluate the analytic expression directly instead of splining.
- The transfer function is on an extremely coarse or irregular grid and you need accuracy beyond what linear splines in log-space provide (hmf offers extrapolate_with_eh=True using the Eisenstein & Hu fitting formula for high-k extrapolation as an alternative).

## Inputs

- CAMB transfer function T(k) (from pycamb or a CAMB-format file)
- Unnormalised power spectrum P0(k) = k^n * T^2 to be re-gridded
- k-bounds defining the re-grid range (e.g. default hmf k-limits ~1e-8 to 2000 h/Mpc)
- Number of grid steps (4097) for the re-gridding

## Outputs

- Log-space linearly-spline-interpolated, uniformly re-gridded P(k) array over the k-bounds
- Interpolated integrand suitable for Romberg integration of the mass variance sigma(M)
- Interpolated/extrapolated cumulative HMF n(>M)

## How to apply

First obtain the transfer function T(k) from CAMB (via pycamb) or read a CAMB-format file, as hmf accepts either. Build the unnormalised power spectrum P0(k) = k^n * T^2, then interpolate with linear splines on the function in log-space (i.e. interpolate log T / log P0 versus log k) and re-grid the function between the k-bounds in 4097 steps, mirroring hmf's internal scheme. Use this re-gridded representation as the input to subsequent integrals, e.g. the top-hat window integrand W(x)=3(sin x - x cos x)/x^3 integrated with Romberg quadrature over [0, kR] and [kR, inf). The same linear-spline-in-log-space approach is used for the cumulative HMF n(>M) interpolation and extrapolation. Rationale: linear splines on log-space quantities avoid spurious oscillations that higher-order splines can introduce into the variance integral, and the dense uniform re-grid makes Romberg integration deterministic and comparable across parameter settings.

## Related tools

- **hmf** (Reference implementation whose interpolation scheme (linear splines on log-space, 4097-step re-grid) is being reproduced) — https://github.com/steven-murray/HMF
- **CAMB** (Generates the transfer functions that are interpolated in log-space) — https://github.com/joezuntz/pycamb
- **Python** (Language in which the spline interpolation and re-gridding are implemented)

## Examples

```
from scipy.interpolate import interp1d; import numpy as np; f = interp1d(np.log(k), np.log(P0), kind='linear', bounds_error=False); kgrid = np.logspace(np.log10(k_min), np.log10(k_max), 4097); P0_grid = np.exp(f(np.log(kgrid)))
```

## Evaluation signals

- The interpolated P(k) evaluated at the original CAMB grid points reproduces the input T(k) values to within spline (linear-in-log) interpolation error.
- The re-grid uses exactly 4097 steps between the k-bounds, matching hmf's internal grid.
- Romberg-integrated sigma(M) from the splined P(k) converges: the partial integrals [0, kR] and [kR, inf) retain 95% of the total for kR satisfying k_min*R_max < 0.1 and k_max*R_min > 3.
- hmf issues no k-bounds warning for the default k-limits (~1e-8 to 2000 h/Mpc), indicating the interpolation grid supports converged integrals.

## Limitations

- Linear splines in log-space assume the transfer function is well sampled; undersampled high-k regions may need extrapolate_with_eh=True (Eisenstein & Hu fitting formula) instead.
- The 4097-step re-grid and Romberg scheme tie the result to hmf's specific quadrature; changing either changes convergence behaviour of the kR bound check (k_min*R_max < 0.1, k_max*R_min > 3).

## Evidence

- [methods] Cumulative HMF interpolation: [evidence span withheld — anchor status: not_found]
- [methods] Open-source reference implementation: [evidence span withheld — anchor status: not_found]
