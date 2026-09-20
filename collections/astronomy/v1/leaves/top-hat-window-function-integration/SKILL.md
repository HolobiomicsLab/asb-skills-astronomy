---
name: top-hat-window-function-integration
description: Use when auditing or reproducing how hmf ensures convergence of the σ(M)
  window-function integrals, or when validating whether a chosen k-range in h/Mpc
  satisfies the kR bound rule for a given halo-mass range.
when_to_use_negative:
- The goal is the HMF fitting-function evaluation or the cumulative HMF (ngtm/dndlnm)
  itself rather than verifying convergence of the σ(M) integrals
- The transfer function is not in CAMB format and no CAMB run is available — the interpolation-and-regridding
  scheme assumes CAMB-format input
- A non-top-hat window function is being studied; the 95% kR rule derived here is
  specific to W(x) = 3(sin x − x cos x)/x³
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: hmf
  role: Reference implementation whose integration scheme (linear spline interpolation
    in log-space, Romberg integration with 4097 steps) and tools.py kR bound warning
    are being reproduced
  repo: https://github.com/steven-murray/HMF
- name: CAMB
  role: Computes the fiducial matter transfer functions used to build the unnormalised
    power spectrum P0(k)
  repo: https://github.com/joezuntz/pycamb
- name: Python
  role: Implementation language for the reconstruction script (splines, Romberg integration,
    mass scan, CSV output)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2013.11.001
    title: 'HMFcalc: An online tool for calculating dark matter halo mass functions'
  source_content_hash: sha256:e7bd0f24e14993bde616b77fa1aab47df4c1ad5611730ddf60b1bdf6f256b4ca
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/steven-murray/HMF
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
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# top-hat-window-function-integration

## Summary

Reconstructs hmf's numerical evaluation of the mass variance σ(M) by integrating the top-hat window function W(x) = 3(sin x − x cos x)/x³ against the unnormalised power spectrum P0(k) = k^n T², and derives the kR convergence bounds (k_min·R_max < 0.1, k_max·R_min > 3) from a 95% integral-fraction criterion.

## When to use

Use when auditing or reproducing how hmf ensures convergence of the σ(M) window-function integrals, or when validating whether a chosen k-range in h/Mpc satisfies the kR bound rule for a given halo-mass range.

## When NOT to use

- The goal is the HMF fitting-function evaluation or the cumulative HMF (ngtm/dndlnm) itself rather than verifying convergence of the σ(M) integrals
- The transfer function is not in CAMB format and no CAMB run is available — the interpolation-and-regridding scheme assumes CAMB-format input
- A non-top-hat window function is being studied; the 95% kR rule derived here is specific to W(x) = 3(sin x − x cos x)/x³

## Inputs

- Fiducial matter transfer function T(k) from CAMB (via pycamb) or a pre-calculated CAMB-format transfer function file shipped with hmf
- Spectral index n for the unnormalised power spectrum P0(k) = k^n * T^2
- Halo mass range (or equivalently top-hat radii R) to scan
- hmf default k-limits (~1e-8 to 2000 h/Mpc)

## Outputs

- Per-mass kR cutoff values at which partial integrals retain 95% of the total mass-variance integral
- Integral-fraction table for the [0, kR] and [kR, inf) partial integrals per scanned R
- Pass/fail flags for the bound checks k_min*R_max < 0.1 and k_max*R_min > 3, written to kR_convergence_results.csv

## How to apply

Obtain a fiducial transfer function with CAMB via pycamb (or use the pre-calculated transfer function shipped with hmf), then build the unnormalised power spectrum P0(k) = k^n * T^2, interpolating it with linear splines in log-space and re-gridding between the k-bounds in 4097 steps exactly as hmf does. Define the top-hat window W(x) = 3(sin x − x cos x)/x³ and numerically integrate the mass-variance integrand over [0, kR] and [kR, inf) using Romberg integration with 4097 steps, mirroring hmf's integration scheme. Scan halo masses (i.e. radii R) to locate, for each R, the kR cutoffs at which the partial integrals retain 95% of the total integral — this 95% rule is what generates the rule of thumb k_min*R_max < 0.1 and k_max*R_min > 3. Finally check whether the default hmf k-limits (~1e-8 to 2000 h/Mpc) satisfy both inequalities, reproducing the behaviour of the tools.py function that warns when the kR bounds are violated.

## Related tools

- **hmf** (Reference implementation whose integration scheme (linear spline interpolation in log-space, Romberg integration with 4097 steps) and tools.py kR bound warning are being reproduced) — https://github.com/steven-murray/HMF
- **CAMB** (Computes the fiducial matter transfer functions used to build the unnormalised power spectrum P0(k)) — https://github.com/joezuntz/pycamb
- **Python** (Implementation language for the reconstruction script (splines, Romberg integration, mass scan, CSV output))

## Examples

```
python kR_convergence_check.py --transfer_fn camb_transfer.dat --n 1.0 --kmin 1e-8 --kmax 2000 --steps 4097 --output kR_convergence_results.csv
```

## Evaluation signals

- The 95% integral-fraction criterion is met for every scanned halo mass R at the located kR cutoffs
- Both kR bound inequalities hold for the default k-limits (~1e-8 to 2000 h/Mpc): k_min*R_max < 0.1 and k_max*R_min > 3
- The re-gridding of P0(k) between k-bounds uses exactly 4097 steps, matching hmf's scheme
- Partial integrals use Romberg integration with 4097 steps, matching hmf's integration scheme
- Reproducing the hmf tools.py bound check on a violating k-range correctly raises the same warning behaviour as hmf's default

## Limitations

- The derived kR bounds are a 95% rule of thumb, not an exact convergence guarantee; integrals retaining slightly less than 95% may still be acceptable for some applications
- The bound check is tied to hmf's specific interpolation (linear splines in log-space) and Romberg integration with 4097 steps; other numerics may shift the cutoffs
- Transfer functions must be produced by CAMB or be in CAMB format, and if not provided the code relies on extrapolation options (e.g. extrapolate_with_eh) that can affect high-k behaviour

## Evidence

- [other] Define the top-hat window function W(x) = 3(sin x - x cos x)/x^3 and numerically integrate the mass-variance integrand over [0, kR] and [kR, inf) using Romberg integration with 4097 steps, mirroring hmf's integration scheme.: [evidence span withheld — anchor status: not_found]
- [other] The kR convergence bounds derived from the 95% rule (giving k_min*R_max < 0.1 and k_max*R_min > 3) are built into hmf by default, which issues a warning if they are not obeyed.: [evidence span withheld — anchor status: not_found]
- [readme] The whole code has been made open-source at https://github.com/steven-murray/HMF.: [evidence span withheld — anchor status: not_found]
