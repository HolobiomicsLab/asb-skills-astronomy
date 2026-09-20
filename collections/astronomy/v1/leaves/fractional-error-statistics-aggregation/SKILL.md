---
name: fractional-error-statistics-aggregation
description: Use when validating a fast analytic approximation or emulator against
  a slower trusted solver by summarizing its accuracy as a single RMS fractional error
  across sampled inputs.
when_to_use_negative:
- When the accuracy target is the full linear matter power spectrum P(k) over k =
  9e-3 - 9 h Mpc^-1 rather than the scalar integrated quantity sigma8 — that uses
  the separate 0.2% RMS P(k) metric
- When a trusted reference solver is unavailable or the approximation is evaluated
  outside the cosmology range it was trained on — fractional errors against a mismatched
  reference are not meaningful
tools:
- name: numpy
  role: Computes the per-cosmology fractional errors and the root-mean-squared aggregation
- name: CAMB
  role: Provides the trusted reference sigma8 from the Boltzmann-equation solution
    for each sampled cosmology
- name: symbolic_pofk
  role: Supplies the analytic sigma8 expression as simple Python functions evaluated
    with numpy
  repo: github.com/DeaglanBartlett/symbolic_pofk
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202348811
    title: A precise symbolic emulator of the linear matter power spectrum
  source_content_hash: sha256:44257b61f1b173726ca0dfeec855fb99216a60faa372df6a3a46479af9e9b58a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAMB
  - numpy
  - symbolic_pofk
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# fractional-error-statistics-aggregation

## Summary

Compute per-sample fractional errors against a trusted reference (here, analytic sigma8 vs CAMB Boltzmann-equation solutions) and aggregate them into a root-mean-squared fractional error, the accuracy metric the paper uses to certify its symbolic emulators.

## When to use

Use when validating a fast analytic approximation or emulator against a slower trusted solver by summarizing its accuracy as a single RMS fractional error across sampled inputs.

## When NOT to use

- When the accuracy target is the full linear matter power spectrum P(k) over k = 9e-3 - 9 h Mpc^-1 rather than the scalar integrated quantity sigma8 — that uses the separate 0.2% RMS P(k) metric
- When a trusted reference solver is unavailable or the approximation is evaluated outside the cosmology range it was trained on — fractional errors against a mismatched reference are not meaningful

## Inputs

- Sampled cosmological parameter sets spanning the emulator's Lambda-CDM training domain (as used for arXiv:2311.15865)
- Analytic sigma8 values from the symbolic_pofk Python emulator (numpy)
- Reference sigma8 values from CAMB Boltzmann-equation runs per cosmology

## Outputs

- Per-cosmology table of fractional errors (analytic sigma8 - CAMB sigma8)/CAMB sigma8 written to sigma8_rms_error.csv
- Aggregate RMS fractional error summary in sigma8_rms_error.csv

## How to apply

Sample Lambda-CDM cosmological parameters across the same domain used to train the Bartlett et al. 2023 linear P(k) emulator, then evaluate the paper's analytic sigma8 expression via the symbolic_pofk Python emulator (numpy) for every sampled cosmology. Run CAMB for each cosmology to obtain reference sigma8 from the Boltzmann-equation solution. For each cosmology compute the fractional error (analytic sigma8 - CAMB sigma8)/CAMB sigma8, then aggregate with numpy into the root-mean-squared fractional error sqrt(mean(err^2)). Write the per-cosmology table and RMS summary to sigma8_rms_error.csv; the published validation threshold is an RMS fractional error of 0.1%. The same fractional-error convention also reports maximum fractional error alongside RMS, as in the paper's error tables.

## Related tools

- **numpy** (Computes the per-cosmology fractional errors and the root-mean-squared aggregation)
- **CAMB** (Provides the trusted reference sigma8 from the Boltzmann-equation solution for each sampled cosmology)
- **symbolic_pofk** (Supplies the analytic sigma8 expression as simple Python functions evaluated with numpy) — github.com/DeaglanBartlett/symbolic_pofk

## Examples

```
import numpy as np; frac_err = (s8_analytic - s8_camb)/s8_camb; rms = np.sqrt(np.mean(frac_err**2))
```

## Evaluation signals

- The RMS fractional error of analytic sigma8 vs CAMB reproduces the published accuracy level of 0.1% across the same cosmology range
- Per-cosmology fractional errors are small and roughly centered on zero, with a reported maximum fractional error alongside the RMS value
- sigma8_rms_error.csv contains both the per-cosmology fractional-error table and the RMS summary
- The expression is 'easily invertible', so solving it for A_s given sigma8 and the other cosmological parameters recovers A_s consistently

## Limitations

- The 0.1% RMS accuracy claim holds only across the same range of cosmologies used for the P(k) emulator; accuracy outside that domain is not established
- RMS fractional error is relative to CAMB as reference, so any CAMB-side settings or convergence issues propagate directly into the metric

## Evidence

- [abstract] Analytic approximation for sigma8 has 0.1% RMS fractional error: [evidence span withheld — anchor status: not_found]
- [results] RMS and maximum fractional errors reported together: "Maximum fractional error: 0.0015%
RMS fractional error: 0.0011%"
- [readme] Emulators are simple Python functions: "Here we give the emulators as simple python functions. 
Some emulators are given as torch and/or fortran90 functions, but all can be 
easily copied, pasted and modified"
- [abstract] Emulator accuracy defined via root mean squared fractional error: [evidence span withheld — anchor status: not_found]
