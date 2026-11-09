---
name: fractional-error-rms-quantification
description: Use when a fast analytic approximation or emulator must be validated
  against a high-fidelity reference solver, and accuracy is reported as pointwise
  fractional difference aggregated to an RMS value.
when_to_use_negative:
- 'When comparing implementations of the same formula (Python vs Fortran): differences
  there are expected only at O(1e-6) and represent implementation agreement, not emulation
  accuracy.'
- When the target is the nonlinear P(k), sigma_8 conversion, or extended cosmologies
  (neutrino masses, w0, wa) — separate emulators and validation ranges apply.
- When evaluating the symbolic emulator outside its trained k range without the Eisenstein
  & Hu extrapolation (extrapolate=False), since the reported 0.2% accuracy applies
  only within the trained range.
tools:
- name: CAMB
  role: Boltzmann-equation reference solver against which the analytic formula's RMS
    fractional error is computed
- name: numpy
  role: Evaluates the analytic linear P(k) on the k grid and aggregates pointwise
    fractional differences into the RMS
- name: Python
  role: Hosts the symbolic emulator functions (plin_emulated, logF_max_precision,
    logF_fiducial) and drives the comparison
  repo: github.com/deaglanbartlett/symbolic_pofk
- name: Fortran 90
  role: Alternative compiled implementation used to bound implementation noise at
    O(1e-6) below the emulation error
  repo: github.com/DeaglanBartlett/symbolic_pofk
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202348811
    title: A precise symbolic emulator of the linear matter power spectrum
  source_content_hash: sha256:b7e632091778d233ea08e0dd79e39d3005bd1ba7a33f646dc26cf45a4953f32d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAMB
  - Fortran 90
  - Python
  - numpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# fractional-error-rms-quantification

## Summary

Quantifies emulation accuracy as the root-mean-squared fractional error between a fast analytic emulator (e.g. the symbolic P(k) formula) and a Boltzmann-code reference (CAMB), aggregated pointwise over a fixed k range (here 9e-3 to 9 h/Mpc at z=0). It reproduces the paper's headline 0.2% RMS fractional accuracy claim.

## When to use

Use when a fast analytic approximation or emulator must be validated against a high-fidelity reference solver, and accuracy is reported as pointwise fractional difference aggregated to an RMS value.

## When NOT to use

- When comparing implementations of the same formula (Python vs Fortran): differences there are expected only at O(1e-6) and represent implementation agreement, not emulation accuracy.
- When the target is the nonlinear P(k), sigma_8 conversion, or extended cosmologies (neutrino masses, w0, wa) — separate emulators and validation ranges apply.
- When evaluating the symbolic emulator outside its trained k range without the Eisenstein & Hu extrapolation (extrapolate=False), since the reported 0.2% accuracy applies only within the trained range.

## Inputs

- Cosmologies sampled across the LambdaCDM prior (e.g. omega_cdm, omega_baryon, A_s, h, ns)
- Grid of wavenumbers k from 9e-3 to 9 h/Mpc at z=0
- Emulated analytic P(k) from plin_emulated()/logF_max_precision()/logF_fiducial()
- Reference linear P(k) from CAMB for the same cosmologies and k grid

## Outputs

- Pointwise fractional-difference array (emulated vs CAMB)
- Root-mean-squared fractional error over k = 9e-3 to 9 h/Mpc
- Comparison table (k, emulated P(k), CAMB P(k), fractional error) written to linear_pk_rms_error.csv

## How to apply

Sample cosmologies spanning the ΛCDM prior and evaluate the symbolic analytic linear P(k) (via plin_emulated(), logF_max_precision() and logF_fiducial(), with default Eisenstein & Hu extrapolation including baryons outside the trained k range) on a numpy grid of k from 9e-3 to 9 h/Mpc at z=0. Run CAMB — the Boltzmann-equation reference, against which the analytic formula is ~950 times faster — on the same cosmologies and k grid. Compute pointwise fractional differences (emulated − reference)/reference and aggregate to the RMS across the full k range. The key decision point is separating emulation error from implementation noise: Python-vs-Fortran implementations agree only to O(1e-6), so any validated error must exceed that floor. Write the comparison table (k, emulated P(k), CAMB P(k), fractional error) and headline RMS to a CSV such as linear_pk_rms_error.csv.

## Related tools

- **CAMB** (Boltzmann-equation reference solver against which the analytic formula's RMS fractional error is computed)
- **numpy** (Evaluates the analytic linear P(k) on the k grid and aggregates pointwise fractional differences into the RMS)
- **Python** (Hosts the symbolic emulator functions (plin_emulated, logF_max_precision, logF_fiducial) and drives the comparison) — github.com/deaglanbartlett/symbolic_pofk
- **Fortran 90** (Alternative compiled implementation used to bound implementation noise at O(1e-6) below the emulation error) — github.com/DeaglanBartlett/symbolic_pofk

## Examples

```
git clone git@github.com:DeaglanBartlett/symbolic_pofk.git && cd symbolic_pofk && pip install . && python examples/linear_example.py
```

## Evaluation signals

- RMS fractional error over k = 9e-3 to 9 h/Mpc at z=0 is ~0.2%, matching the value reported in the paper's abstract
- Emulated-vs-CAMB fractional differences exceed the O(1e-6) Python-vs-Fortran implementation agreement, so emulation error dominates rather than implementation noise
- Comparison table contains all four columns (k, emulated P(k), CAMB P(k), fractional error) with matched k grids and identical cosmologies
- Extrapolation handling is consistent: outside the trained k range the Eisenstein & Hu fit (including baryons) is used unless extrapolate=False is set
- Validation is repeated across cosmologies spanning the full LambdaCDM prior, not a single fiducial cosmology

## Limitations

- The 0.2% RMS accuracy holds only within the trained k range (9e-3 to 9 h/Mpc); outside it, accuracy relies on the Eisenstein & Hu fit with baryons, which is a different approximation.
- The 0.2% figure applies to the linear matter power spectrum only; other outputs (sigma_8 at 0.1%, nonlinear P(k), comoving distance, growth factor) have their own accuracies.
- Fractional differences below O(1e-6) between Python and Fortran implementations cannot be interpreted as emulation error.

## Evidence

- [abstract] headline 0.2% RMS: [evidence span withheld — anchor status: not_found]
- [methods] Eisenstein & Hu extrapolation control: "we use the Eisenstein & Hu fit which includes baryons. This can be switched off by setting extrapolate=False"
- [readme] implementation noise floor: "they are identical up to a fractional difference of
O(1e-6), which is much smaller than the error on the emulation"
- [abstract] relative speed vs reference: [evidence span withheld — anchor status: not_found]
- [methods] extrapolation functions: "setting extrapolate=False in the functions plin_emulated(), logF_max_precision() and logF_fiducial()"
