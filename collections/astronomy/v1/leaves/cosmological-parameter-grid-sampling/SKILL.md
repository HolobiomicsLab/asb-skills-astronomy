---
name: cosmological-parameter-grid-sampling
description: Use when benchmarking speed or accuracy of cosmological emulators (e.g.
  symbolic_pofk plin_emulated() vs CAMB vs BACCO) — identical cosmology/k grids are
  required for fair per-point ratio computation.
when_to_use_negative:
- The emulator must be evaluated outside its trained k range or cosmology prior —
  the Eisenstein & Hu extrapolation is a fallback, not a validated grid point
- Comparing emulators trained/validated on different parameter domains without redefining
  a common grid first
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3299
- http://edamontology.org/topic_3318
tools:
- name: symbolic_pofk
  role: Source of plin_emulated() and the validated k range used to define the grid
  repo: https://github.com/DeaglanBartlett/symbolic_pofk
- name: CAMB
  role: Boltzmann-equation reference evaluated on the same cosmology grid for timing
    and error baselines
- name: BACCO
  role: Neural-network-based emulator timed on the identical cosmology/k grid
- name: numpy
  role: Defines the cosmology and wavenumber arrays forming the shared grid
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_004
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202348811
    title: A precise symbolic emulator of the linear matter power spectrum
  source_content_hash: sha256:3c478defe806cdf814459553b0fb4ad8ec9e6dd1cef3fa3015f11189a1cd74c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DeaglanBartlett/symbolic_pofk
  tools:
  - BACCO
  - CAMB
  - PyTorch
  - numpy
  - symbolic_pofk
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3299
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cosmological-parameter-grid-sampling

## Summary

Defines a shared grid of ΛCDM cosmologies and wavenumbers (numpy arrays) spanning the symbolic emulator's validated range k = 9×10⁻³ to 9 h Mpc⁻¹ at redshift zero, so that symbolic_pofk, CAMB, and BACCO are evaluated on identical inputs for timing and accuracy comparisons.

## When to use

Use when benchmarking speed or accuracy of cosmological emulators (e.g. symbolic_pofk plin_emulated() vs CAMB vs BACCO) — identical cosmology/k grids are required for fair per-point ratio computation.

## When NOT to use

- The emulator must be evaluated outside its trained k range or cosmology prior — the Eisenstein & Hu extrapolation is a fallback, not a validated grid point
- Comparing emulators trained/validated on different parameter domains without redefining a common grid first

## Inputs

- ΛCDM cosmological parameter samples covering the emulator's training/prior range (as in Bartlett et al. 2023, arXiv:2311.15865)
- numpy array of wavenumbers k spanning 9×10⁻³ to 9 h Mpc⁻¹ at redshift zero

## Outputs

- Shared (cosmology, k) evaluation grid consumed identically by symbolic_pofk, CAMB, and BACCO
- Per-point timing/error tables, e.g. benchmark_speedup.csv and sigma8_rms_error.csv with fractional-error columns and RMS summaries

## How to apply

Sample ΛCDM cosmological parameters across the same domain used to train the linear matter power spectrum emulator of Bartlett et al. 2023 (arXiv:2311.15865), and define wavenumbers as a numpy array spanning the emulator's validated range k = 9×10⁻³ to 9 h Mpc⁻¹ at redshift zero. Every method under comparison — plin_emulated() for the symbolic emulator, CAMB Boltzmann-equation runs, and the BACCO neural-network emulator — must be evaluated on this exact same cosmology/k grid, otherwise per-point speedup and error ratios are not meaningful. The same grid also supports RMS fractional-error validation: e.g. computing (analytic sigma8 − CAMB sigma8)/CAMB sigma8 per cosmology and aggregating to an RMS, which should be ~0.1% for the As-to-sigma8 conversion and ~0.2% for the linear P(k) emulator. For extrapolation outside the trained k range, the package defaults to the Eisenstein & Hu fit with baryons (extrapolate=True); set extrapolate=False to restrict strictly to the validated range.

## Related tools

- **symbolic_pofk** (Source of plin_emulated() and the validated k range used to define the grid) — https://github.com/DeaglanBartlett/symbolic_pofk
- **CAMB** (Boltzmann-equation reference evaluated on the same cosmology grid for timing and error baselines)
- **BACCO** (Neural-network-based emulator timed on the identical cosmology/k grid)
- **numpy** (Defines the cosmology and wavenumber arrays forming the shared grid)

## Examples

```
git clone git@github.com:DeaglanBartlett/symbolic_pofk.git && cd symbolic_pofk && pip install . && python examples/linear_example.py
```

## Evaluation signals

- Measured speedup ratios on the grid should be of the same order as the reported values: ~950x versus CAMB and ~36x versus BACCO
- Symbolic emulator's RMS fractional accuracy over the grid should be ~0.2% for k = 9×10⁻³ to 9 h Mpc⁻¹
- Analytic sigma8 conversion should reproduce ~0.1% RMS fractional error against CAMB across the sampled cosmologies
- All compared methods must be evaluated at every (k, cosmology) point of the identical grid — missing points invalidate the ratios
- Benchmark script runs end-to-end (script_runs) and writes timings/ratios to benchmark_speedup.csv

## Limitations

- Grid validity is bounded by the emulator's trained range; outside k = 9×10⁻³ to 9 h Mpc⁻¹ the code silently substitutes the Eisenstein & Hu fit with baryons unless extrapolate=False is set
- Speedup ratios are order-of-magnitude checks; exact values depend on hardware and the chosen grid density

## Evidence

- [methods] Define a common evaluation grid: [evidence span withheld — anchor status: not_found]
- [abstract] Range and accuracy anchor: [evidence span withheld — anchor status: not_found]
- [methods] Training-domain scope: [evidence span withheld — anchor status: not_found]
- [readme] Extrapolation fallback: "we use the Eisenstein & Hu fit which includes baryons. This can be switched off by setting `extrapolate=False` in the functions `plin_emulated()`"
