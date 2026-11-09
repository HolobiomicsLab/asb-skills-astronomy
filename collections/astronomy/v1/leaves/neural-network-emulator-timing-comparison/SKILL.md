---
name: neural-network-emulator-timing-comparison
description: Use when a study needs to quantify computational cost of power-spectrum
  evaluation — e.g. justifying an analytic emulator for MCMC or large surveys by timing
  it against CAMB and BACCO on the same grid.
when_to_use_negative:
- The goal is accuracy assessment rather than speed — use the paper's RMS fractional
  error analysis (0.2% over k = 9e-3 to 9 h/Mpc) instead of timing benchmarks.
- The target is the nonlinear power spectrum or extended cosmologies (neutrino masses,
  w0, wa), which are covered by separate emulators (halofit_example.py, syren_new_example.ipynb)
  not by the linear LCDM timing comparison.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: symbolic_pofk
  role: Provides the analytic linear power spectrum emulator plin_emulated() being
    timed, installed via pip from the cloned repository
  repo: github.com/DeaglanBartlett/symbolic_pofk
- name: CAMB
  role: Boltzmann-equation reference code whose linear power spectrum evaluation time
    provides the baseline for the ~950x speedup ratio
- name: BACCO
  role: Neural-network-based matter power spectrum emulator whose evaluation time
    provides the baseline for the ~36x speedup ratio
- name: numpy
  role: Defines the common cosmology and wavenumber evaluation grid as numpy arrays
- name: Python
  role: Timing protocol via time.perf_counter and example scripts such as examples/linear_example.py
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202348811
    title: A precise symbolic emulator of the linear matter power spectrum
  source_content_hash: sha256:22cde0ae85a0689dfd2c05e12b517425e66d5855fc7a7382380d2333f5408424
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BACCO
  - CAMB
  - Python
  - numpy
  - symbolic_pofk
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# neural-network-emulator-timing-comparison

## Summary

Benchmarks the evaluation speed of the symbolic analytic linear power spectrum emulator (plin_emulated from symbolic_pofk) against the Boltzmann code CAMB and the neural-network-based emulator BACCO on a common cosmology/k grid, quantifying the reported ~950x and ~36x speedups.

## When to use

Use when a study needs to quantify computational cost of power-spectrum evaluation — e.g. justifying an analytic emulator for MCMC or large surveys by timing it against CAMB and BACCO on the same grid.

## When NOT to use

- The goal is accuracy assessment rather than speed — use the paper's RMS fractional error analysis (0.2% over k = 9e-3 to 9 h/Mpc) instead of timing benchmarks.
- The target is the nonlinear power spectrum or extended cosmologies (neutrino masses, w0, wa), which are covered by separate emulators (halofit_example.py, syren_new_example.ipynb) not by the linear LCDM timing comparison.

## Inputs

- Fiducial set of ΛCDM cosmologies (cosmological parameter arrays)
- Wavenumber grid as numpy arrays spanning k = 9e-3 to 9 h/Mpc at redshift zero
- symbolic_pofk package installed via pip (git clone, pip install .)

## Outputs

- benchmark_speedup.csv containing evaluation timings and speedup ratios for symbolic emulator, CAMB, and BACCO
- Mean evaluation times per emulator measured with time.perf_counter
- Speedup ratios (~950x vs CAMB, ~36x vs BACCO)

## How to apply

Install the symbolic_pofk emulators with pip so plin_emulated() and examples/linear_example.py are available. Define a common evaluation grid of fiducial ΛCDM cosmologies with wavenumbers spanning the emulator's validated range k = 9e-3 to 9 h/Mpc at redshift zero as numpy arrays. Evaluate the symbolic emulator, CAMB, and the neural-network BACCO emulator on this identical grid, timing repeated evaluations with Python's time.perf_counter to obtain robust mean times per code. Compute per-point speedup ratios (CAMB time / symbolic time and BACCO time / symbolic time) and write timings plus ratios to benchmark_speedup.csv. Ratios should be of the same order as the published values (~950x vs CAMB, ~36x vs BACCO); accuracy is confirmed by the emulator's 0.2% RMS fractional error over the same k range.

## Related tools

- **symbolic_pofk** (Provides the analytic linear power spectrum emulator plin_emulated() being timed, installed via pip from the cloned repository) — github.com/DeaglanBartlett/symbolic_pofk
- **CAMB** (Boltzmann-equation reference code whose linear power spectrum evaluation time provides the baseline for the ~950x speedup ratio)
- **BACCO** (Neural-network-based matter power spectrum emulator whose evaluation time provides the baseline for the ~36x speedup ratio)
- **numpy** (Defines the common cosmology and wavenumber evaluation grid as numpy arrays)
- **Python** (Timing protocol via time.perf_counter and example scripts such as examples/linear_example.py)

## Examples

```
git clone git@github.com:DeaglanBartlett/symbolic_pofk.git; cd symbolic_pofk; pip install .; python examples/linear_example.py
```

## Evaluation signals

- Measured CAMB/symbolic speedup ratio is of the same order as the reported ~950x
- Measured BACCO/symbolic speedup ratio is of the same order as the reported ~36x
- The symbolic emulator evaluated is the correct one, confirmed by its 0.2% RMS fractional accuracy over k = 9e-3 to 9 h/Mpc
- benchmark_speedup.csv is produced with per-point timings and ratios for all three evaluators on the identical grid
- All evaluations stay within the emulator's validated k range (or extrapolate=False is set to disable the Eisenstein & Hu fallback)

## Limitations

- The speedups are specific to the validated k range of 9e-3 to 9 h/Mpc at redshift zero; outside this range the emulator falls back to the Eisenstein & Hu fit unless extrapolate=False.
- Timing comparisons depend on the chosen timing protocol (repeated evaluations with time.perf_counter); CAMB and BACCO must be timed on the identical cosmology/k grid for ratios to be meaningful.

## Evidence

- [abstract] Analytic approximation is 950 times faster to evaluate than CAMB and 36 times faster than BACCO: [evidence span withheld — anchor status: not_found]
- [abstract] RMS fractional error of 0.2 percent over validated k range: [evidence span withheld — anchor status: not_found]
- [methods] Install emulators via pip: [evidence span withheld — anchor status: not_found]
- [methods] Run the LCDM linear emulator example: "We give an example for how to use the ΛCDM linear emulator in examples/linear_example.py."
- [readme] Eisenstein & Hu extrapolation outside validated k range: "outside the k range tested in Bartlett et al. 2023, we use the Eisenstein & Hu fit which includes baryons. This can be switched off by setting `extrapolate=False`"
