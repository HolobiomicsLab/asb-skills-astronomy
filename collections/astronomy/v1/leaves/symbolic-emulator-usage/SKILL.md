---
name: symbolic-emulator-usage
description: Use when a pipeline needs As from sigma8 (or vice versa) for LambdaCDM
  cosmologies and needs sub-percent accuracy without running CAMB, e.g. setting initial
  conditions from sigma8-matched simulations.
when_to_use_negative:
- The cosmology includes extended parameters (massive neutrinos, w0, wa) — the 0.1%
  sigma8 conversion is reported for the LambdaCDM range; use the SYREN-NEW emulators
  instead
- The goal is the linear or nonlinear matter power spectrum P(k) itself rather than
  the sigma8 <-> As scalar conversion — use plin_emulated() or the halofit emulator
  instead
- Accuracy beyond the published prior range is required, where the emulators' tested-domain
  extrapolation (Eisenstein & Hu fit with baryons, switchable via extrapolate=False)
  is not validated
tools:
- name: symbolic_pofk
  role: Provides the analytic, invertible sigma8 <-> As conversion functions (linear.sigma8_to_As
    / As_to_sigma8) used for the round-trip test
  repo: github.com/DeaglanBartlett/symbolic_pofk
- name: Python
  role: Language in which the emulators are given as simple functions; used to implement
    the numpy root solver and evaluation grid
  repo: github.com/DeaglanBartlett/symbolic_pofk
- name: numpy
  role: Vectorized evaluation of the analytic sigma8(As) expression over the cosmology
    grid and implementation of the inversion root solver
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202348811
    title: A precise symbolic emulator of the linear matter power spectrum
  source_content_hash: sha256:8d5e9d6b118bf5df0b23531fa4cbb42b0c35f5e075a6bc16ce1bcfb0ef14cd66
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - numpy
  - symbolic_pofk
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# symbolic-emulator-usage

## Summary

Applies the symbolic_pofk analytic emulators, specifically the easily invertible sigma8(As) conversion, to translate between sigma8 and As for given LambdaCDM cosmologies and to validate the round-trip against the paper's reported 0.1% RMS fractional error.

## When to use

Use when a pipeline needs As from sigma8 (or vice versa) for LambdaCDM cosmologies and needs sub-percent accuracy without running CAMB, e.g. setting initial conditions from sigma8-matched simulations.

## When NOT to use

- The cosmology includes extended parameters (massive neutrinos, w0, wa) — the 0.1% sigma8 conversion is reported for the LambdaCDM range; use the SYREN-NEW emulators instead
- The goal is the linear or nonlinear matter power spectrum P(k) itself rather than the sigma8 <-> As scalar conversion — use plin_emulated() or the halofit emulator instead
- Accuracy beyond the published prior range is required, where the emulators' tested-domain extrapolation (Eisenstein & Hu fit with baryons, switchable via extrapolate=False) is not validated

## Inputs

- Grid of LambdaCDM cosmological parameter vectors (e.g. Omega_m, ns, sigma8) sampled within the paper's published prior range
- Installed symbolic_pofk package (pip install .) providing linear.sigma8_to_As / As_to_sigma8

## Outputs

- Per-cosmology fractional round-trip errors (sigma8 -> As -> sigma8) for each sampled cosmology
- RMS and maximum round-trip fractional error summary values
- sigma8_as_roundtrip_errors.csv containing the per-cosmology and summary statistics

## How to apply

Install the emulators by cloning github.com/DeaglanBartlett/symbolic_pofk and running 'pip install .' from the cloned directory, then import the sigma8 <-> As conversion from the installed package following the conventions in examples/linear_example.py. Sample a numpy grid of LambdaCDM cosmologies (e.g. Omega_m, ns, sigma8) across the published prior range and evaluate the forward analytic sigma8(As) expression for each point. Numerically invert the expression for each cosmology using a numpy root solver (Newton iteration or bisection on As) with a tolerance far tighter than the 0.1% model-level error, so pure numerical residuals stay at O(1e-6). Re-evaluate the forward conversion at the recovered As and record the per-cosmology fractional round-trip difference. Save the fractional errors plus RMS and maximum summaries to a CSV (sigma8_as_roundtrip_errors.csv); validation succeeds if the RMS round-trip error remains at the ~0.1% level reported for the sigma8 conversion across the cosmology range, with numerical residuals no larger than O(1e-6) — much smaller than the error on the emulation.

## Related tools

- **symbolic_pofk** (Provides the analytic, invertible sigma8 <-> As conversion functions (linear.sigma8_to_As / As_to_sigma8) used for the round-trip test) — github.com/DeaglanBartlett/symbolic_pofk
- **Python** (Language in which the emulators are given as simple functions; used to implement the numpy root solver and evaluation grid) — github.com/DeaglanBartlett/symbolic_pofk
- **numpy** (Vectorized evaluation of the analytic sigma8(As) expression over the cosmology grid and implementation of the inversion root solver)

## Examples

```
from symbolic_pofk.linear import sigma8_to_As, As_to_sigma8; As = sigma8_to_As(sigma8, Omega_m, ns)
```

## Evaluation signals

- RMS round-trip fractional error across the cosmology grid remains at the ~0.1% level reported for the sigma8 conversion
- Pure numerical residuals from the inversion solver are no larger than O(1e-6), much smaller than the emulation error
- Maximum round-trip fractional error stays consistent with the 0.1% RMS error across the full prior range, with no outliers from solver failure
- Forward sigma8(As) evaluated at recovered As matches the original sigma8 for every cosmology in the grid
- Output CSV contains per-cosmology fractional errors plus RMS and maximum summary values for all sampled cosmologies

## Limitations

- The 0.1% RMS fractional error is a model-level error, so tightening the inversion solver tolerance below that does not improve overall accuracy
- Accuracy is only demonstrated across the paper's published prior range of LambdaCDM cosmologies; outside it the result is unvalidated
- The conversion was validated for LambdaCDM only; extended cosmologies (neutrino masses, w0, wa) require the SYREN-NEW emulators

## Evidence

- [abstract] Analytic approximation for sigma8 has a root mean squared fractional error of 0.1% across the same range of cosmologies and is invertible to obtain As: [evidence span withheld — anchor status: not_found]
- [methods] Install the emulators and dependencies via pip: [evidence span withheld — anchor status: not_found]
- [readme] Numerical residuals from Python/Fortran implementations bounded at O(1e-6): "they are identical up to a fractional difference of O(1e-6), which is much smaller than the error on the emulation"
- [intro] Symbolic emulator for conversion between sigma8 and As as a function of cosmology: "* Conversion $\sigma_8 \leftrightarrow A_{\rm s}$
as a function of cosmology."
