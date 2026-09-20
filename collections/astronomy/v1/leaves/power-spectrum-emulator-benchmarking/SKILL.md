---
name: power-spectrum-emulator-benchmarking
description: Use when you need to quantify the wall-clock evaluation cost of an analytic
  power-spectrum emulator relative to Boltzmann-solver (CAMB) or neural-network emulator
  (BACCO) references, using identical cosmologies and wavenumbers across k = 9e-3
  to 9 h/Mpc.
when_to_use_negative:
- The goal is to validate emulator accuracy (RMS fractional error ~0.2%) rather than
  timing — accuracy validation requires comparing against Boltzmann-equation solutions,
  not stopwatch benchmarks.
- Evaluations fall outside the validated k range 9e-3 to 9 h/Mpc, where plin_emulated()
  defaults to the lower-accuracy Eisenstein & Hu extrapolation (extrapolate=True).
- The comparison target is the nonlinear (halofit) or extended-cosmology (neutrino
  masses, w0, wa) emulators, whose benchmark numbers differ from the linear ΛCDM emulator
  reported here.
edam_operation: http://edamontology.org/operation_3860
edam_topics:
- http://edamontology.org/topic_3399
- http://edamontology.org/topic_3315
tools:
- name: symbolic_pofk
  role: Provides plin_emulated(), the analytic emulator being timed; installed via
    git clone + pip install .
  repo: github.com/DeaglanBartlett/symbolic_pofk
- name: CAMB
  role: Boltzmann-equation reference linear power spectrum solver timed for the speedup
    comparison (~950x slower than the symbolic emulator)
- name: BACCO
  role: Neural-network-based matter power spectrum emulator timed for comparison (~36x
    slower than the symbolic emulator)
- name: Python
  role: Implementation language; time.perf_counter used for robust timing of repeated
    evaluations
- name: numpy
  role: Used to define the cosmology and wavenumber evaluation grids as arrays
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202348811
    title: A precise symbolic emulator of the linear matter power spectrum
  source_content_hash: sha256:53a8cf6dab30254a54b571f9e1d03407610f2f813d85b0663c9aed6e4532d039
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BACCO
  - CAMB
  - Python
  - numpy
  - symbolic_pofk
  edam_operation: http://edamontology.org/operation_3860
  edam_topics:
  - http://edamontology.org/topic_3399
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# power-spectrum-emulator-benchmarking

## Summary

Benchmark the evaluation speed of the symbolic_pofk analytic linear power spectrum emulator (plin_emulated()) against CAMB and the BACCO neural-network emulator on a common cosmology/k grid, quantifying the reported speedups of ~950x versus CAMB and ~36x versus BACCO.

## When to use

Use when you need to quantify the wall-clock evaluation cost of an analytic power-spectrum emulator relative to Boltzmann-solver (CAMB) or neural-network emulator (BACCO) references, using identical cosmologies and wavenumbers across k = 9e-3 to 9 h/Mpc.

## When NOT to use

- The goal is to validate emulator accuracy (RMS fractional error ~0.2%) rather than timing — accuracy validation requires comparing against Boltzmann-equation solutions, not stopwatch benchmarks.
- Evaluations fall outside the validated k range 9e-3 to 9 h/Mpc, where plin_emulated() defaults to the lower-accuracy Eisenstein & Hu extrapolation (extrapolate=True).
- The comparison target is the nonlinear (halofit) or extended-cosmology (neutrino masses, w0, wa) emulators, whose benchmark numbers differ from the linear ΛCDM emulator reported here.

## Inputs

- Fiducial set of ΛCDM cosmologies (numpy arrays of cosmological parameters)
- Wavenumber grid spanning k = 9e-3 to 9 h/Mpc at redshift zero
- symbolic_pofk emulator installed via pip (plin_emulated() available)
- CAMB and BACCO emulators available as reference evaluators

## Outputs

- Mean evaluation times for symbolic emulator, CAMB, and BACCO (from time.perf_counter over repeated evaluations)
- Per-point speedup ratios (CAMB/symbolic and BACCO/symbolic)
- benchmark_speedup.csv containing the timings and speedup ratios

## How to apply

Install the symbolic_pofk emulators per the README (git clone, cd symbolic_pofk, pip install .) and run examples/linear_example.py as the reference for plin_emulated() usage. Define a fiducial set of ΛCDM cosmologies as numpy arrays, with wavenumbers spanning the emulator's validated range k = 9e-3 to 9 h/Mpc at redshift zero. Evaluate plin_emulated() on this grid, timing repeated calls with Python's time.perf_counter to obtain a robust mean evaluation time, then time CAMB and BACBO evaluations on the identical grid with the same protocol. Compute speedup ratios (reference time divided by symbolic-emulator time) per point and write timings and ratios to benchmark_speedup.csv. Set extrapolate=False in plin_emulated() to restrict to the trained range rather than the Eisenstein & Hu fit used by default outside it, and sanity-check that measured ratios are of the same order as the reported ~950x (CAMB) and ~36x (BACCO) values.

## Related tools

- **symbolic_pofk** (Provides plin_emulated(), the analytic emulator being timed; installed via git clone + pip install .) — github.com/DeaglanBartlett/symbolic_pofk
- **CAMB** (Boltzmann-equation reference linear power spectrum solver timed for the speedup comparison (~950x slower than the symbolic emulator))
- **BACCO** (Neural-network-based matter power spectrum emulator timed for comparison (~36x slower than the symbolic emulator))
- **Python** (Implementation language; time.perf_counter used for robust timing of repeated evaluations)
- **numpy** (Used to define the cosmology and wavenumber evaluation grids as arrays)

## Examples

```
git clone git@github.com:DeaglanBartlett/symbolic_pofk.git; cd symbolic_pofk; pip install .; python examples/linear_example.py
```

## Evaluation signals

- Measured CAMB/symbolic speedup ratio is of the same order as the reported ~950x
- Measured BACCO/symbolic speedup ratio is of the same order as the reported ~36x
- The symbolic emulator used achieves RMS fractional accuracy of 0.2% over k = 9e-3 to 9 h/Mpc, confirming the correct function is being timed
- All three emulators were evaluated on the identical cosmology and wavenumber grid with the same timing protocol (time.perf_counter, repeated evaluations)
- benchmark_speedup.csv is written and contains per-point timings and ratios

## Limitations

- Speedup ratios are validated only over the emulator's trained range k = 9e-3 to 9 h/Mpc; outside this range plin_emulated() uses the Eisenstein & Hu fit (unless extrapolate=False is set), which is not the symbolic expression itself.
- Timing results are hardware- and implementation-dependent; only the order of magnitude of the ratios should be compared to the reported values.
- The 950x/36x figures apply to the redshift-zero linear ΛCDM emulator; the nonlinear, extended-cosmology, baryonic, and wider-prior emulators in the repository have different accuracy and speed characteristics.

## Evidence

- [abstract] Reported speedups vs CAMB and BACCO: [evidence span withheld — anchor status: not_found]
- [abstract] Validated accuracy range of the timed function: [evidence span withheld — anchor status: not_found]
- [methods] Emulator installation procedure: [evidence span withheld — anchor status: not_found]
- [readme] Extrapolation switch controlling what is timed: "setting `extrapolate=False` in the functions `plin_emulated()`, `logF_max_precision()` and `logF_fiducial()`"
- [readme] Reference example for emulator usage: "We give an example for how to use the ΛCDM
linear emulator in `examples/linear_example.py`."
