---
name: romberg-numerical-integration-convergence-testing
description: 'Use when a halo mass function or σ(M) calculation must be verified for
  integral convergence: i.e. when you need to check whether chosen k-bounds and kR
  ranges capture ≥95% of the window-function integral before trusting the HMF output.'
when_to_use_negative:
- When a validated transfer function file is already supplied and hmf's default k-bounds
  are used unchanged and no convergence warning is raised — the check is redundant
- When the power spectrum is normalised via sigma_8 and non-top-hat filters (e.g.
  WDM-corrected T_WDM(k) and non-sharp-k windows) are the object of study; the 95%
  rule here is derived specifically for the top-hat window W(x) = 3(sin x - x cos
  x)/x^3
- When the goal is a differential HMF fitting-function comparison (f(sigma) choice)
  rather than verifying the numerical integrals feeding σ(M)
tools:
- name: hmf
  role: 'Provides the integration scheme being reconstructed: linear-spline log-space
    interpolation, 4097-step re-gridding, Romberg integration of the mass-variance
    integrand, and the tools.py function that warns when the kR bounds are violated'
  repo: https://github.com/steven-murray/HMF
- name: pycamb
  role: Generates the fiducial CAMB transfer function used as input for building P0(k)
    = k^n * T^2
  repo: https://github.com/joezuntz/pycamb
- name: CAMB
  role: Underlying transfer-function computation engine; hmf reads its output file
    format directly
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2013.11.001
    title: 'HMFcalc: An online tool for calculating dark matter halo mass functions'
  source_content_hash: sha256:8a072002e90c41ab441b452e0439b4851eb7050497dcd87a3de890d43a7c67d2
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
yes_rate: 0.9230769230769231
n_claims: 13
---

# Romberg numerical integration convergence testing

## Summary

Reconstructs hmf's kR convergence-bound check on the mass-variance (σ(M)) window-function integrals: numerically integrating the unnormalised power spectrum P0(k)=k^n*T^2 against the top-hat window with Romberg integration, then deriving and testing the 95%-integral-fraction rule that yields k_min*R_max < 0.1 and k_max*R_min > 3.

## When to use

Use when a halo mass function or σ(M) calculation must be verified for integral convergence: i.e. when you need to check whether chosen k-bounds and kR ranges capture ≥95% of the window-function integral before trusting the HMF output.

## When NOT to use

- When a validated transfer function file is already supplied and hmf's default k-bounds are used unchanged and no convergence warning is raised — the check is redundant
- When the power spectrum is normalised via sigma_8 and non-top-hat filters (e.g. WDM-corrected T_WDM(k) and non-sharp-k windows) are the object of study; the 95% rule here is derived specifically for the top-hat window W(x) = 3(sin x - x cos x)/x^3
- When the goal is a differential HMF fitting-function comparison (f(sigma) choice) rather than verifying the numerical integrals feeding σ(M)

## Inputs

- Transfer function T(k,z) in CAMB format (produced by CAMB or read from file), or the pre-calculated transfer function shipped with hmf
- Spectral index n for the unnormalised power spectrum P0(k) = k^n * T^2
- Halo masses / scales R to scan for the kR convergence bounds
- Default k-bounds of the re-gridded power spectrum (~1e-8 to 2000 h/Mpc)

## Outputs

- kR cutoff values at which partial window-function integrals retain 95% of the total, per scanned halo mass R
- Integral-fraction values from the Romberg integration over [0, kR] and [kR, inf)
- Pass/fail flags for the hmf default bounds k_min*R_max < 0.1 and k_max*R_min > 3
- kR_convergence_results.csv with cutoffs, fractions, and bound-check flags

## How to apply

Obtain a fiducial transfer function with CAMB via pycamb (or use the transfer function shipped with hmf), and build the unnormalised power spectrum P0(k) = k^n * T^2, interpolating with linear splines in log-space and re-gridding between k-bounds in 4097 steps as hmf does. Define the top-hat window W(x) = 3(sin x - x cos x)/x^3 and numerically integrate the mass-variance integrand over [0, kR] and [kR, inf) using Romberg integration with 4097 steps, mirroring hmf's integration scheme. Scan halo masses R to locate, for each R, the kR cutoffs at which the partial integrals retain 95% of the total integral. Finally evaluate hmf's default bound condition k_min*R_max < 0.1 and k_max*R_min > 3, reproducing the behaviour of the tools.py function that warns when bounds on kR are violated, and flag any inequality that fails.

## Related tools

- **hmf** (Provides the integration scheme being reconstructed: linear-spline log-space interpolation, 4097-step re-gridding, Romberg integration of the mass-variance integrand, and the tools.py function that warns when the kR bounds are violated) — https://github.com/steven-murray/HMF
- **pycamb** (Generates the fiducial CAMB transfer function used as input for building P0(k) = k^n * T^2) — https://github.com/joezuntz/pycamb
- **CAMB** (Underlying transfer-function computation engine; hmf reads its output file format directly)

## Examples

```
T,E,B,X = pycamb.transfers(2000, H0=70., omegab=0.04, omegav=0.6, omegac=0.36)  # k, T_i(k,z), sigma8(z) for P0(k)=k^n*T^2
```

## Evaluation signals

- The 95% integral-fraction criterion is met for all scanned halo masses R (partial integrals retain ≥95% of the total)
- Both default bound inequalities hold for the default k-limits (~1e-8 to 2000 h/Mpc): k_min*R_max < 0.1 and k_max*R_min > 3
- No convergence warning is emitted by the reconstructed tools.py bound-check for the default k-bounds
- The re-gridding of the interpolated P0(k) uses 4097 steps between k-bounds, matching hmf's scheme
- kR_convergence_results.csv is written and its pass/fail flags are consistent with the computed integral fractions

## Limitations

- The derived bounds (k_min*R_max < 0.1, k_max*R_min > 3) apply to the top-hat window function and the 95% integral-fraction criterion; other filters or accuracy targets would need the scan redone
- The pycamb README states the code is out of date and 'no longer works with recent versions of camb'; modern workflows should use CAMB's built-in Python bindings
- If a provided transfer-function file is not in CAMB format, hmf cannot read it, and extrapolating to high k requires extrapolate_with_eh=True (Eisenstein & Hu fitting formula)

## Evidence

- [other] The kR convergence bounds derived from the 95% rule (giving k_min*R_max < 0.1 and k_max*R_min > 3) are built into hmf by default, which issues a warning if they are not obeyed.: [evidence span withheld — anchor status: not_found]
- [other] Workflow step defining the window and Romberg scheme: [evidence span withheld — anchor status: not_found]
