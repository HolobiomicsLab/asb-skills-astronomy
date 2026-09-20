---
name: boltzmann-reference-spectrum-comparison
description: Use when a fast P(k) emulator or fitting formula must be benchmarked
  for accuracy against a Boltzmann solver, e.g. verifying the ~0.2% RMS fractional
  error over k = 9e-3 to 9 h/Mpc at z=0.
when_to_use_negative:
- You need the nonlinear power spectrum or extended cosmologies (neutrino masses,
  w0, wa) or baryonic effects — use the dedicated halofit, syren-new, or syren-baryon
  emulators/examples instead.
- You only need to check implementation equivalence between the Python and Fortran
  versions, where O(1e-6) fractional agreement — not the ~0.2% emulation error — is
  the relevant tolerance.
- Your k values lie outside 9e-3 to 9 h/Mpc, where only the Eisenstein & Hu extrapolation
  (not the trained symbolic formula) applies.
tools:
- name: CAMB
  role: Boltzmann-equation reference solver; the analytic formula is reported as ~950
    times faster to evaluate than CAMB
- name: numpy
  role: Evaluates the analytic linear P(k) on the k grid for sampled cosmologies
- name: symbolic_pofk
  role: Provides plin_emulated(), logF_max_precision(), logF_fiducial() and the linear_example.py
    script being validated
  repo: github.com/DeaglanBartlett/symbolic_pofk
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202348811
    title: A precise symbolic emulator of the linear matter power spectrum
  source_content_hash: sha256:25cfde2c59871d4f84ecd469099a1b2aeecbcbe18499285fc6c3beb5f2b64ee8
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
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# boltzmann-reference-spectrum-comparison

## Summary

Validates a fast analytic (symbolic) emulator of the redshift-zero linear matter power spectrum by computing pointwise fractional differences against a Boltzmann-equation reference (CAMB) across a sampled cosmology prior and k grid, aggregating to an RMS fractional error to confirm sub-percent accuracy.

## When to use

Use when a fast P(k) emulator or fitting formula must be benchmarked for accuracy against a Boltzmann solver, e.g. verifying the ~0.2% RMS fractional error over k = 9e-3 to 9 h/Mpc at z=0.

## When NOT to use

- You need the nonlinear power spectrum or extended cosmologies (neutrino masses, w0, wa) or baryonic effects — use the dedicated halofit, syren-new, or syren-baryon emulators/examples instead.
- You only need to check implementation equivalence between the Python and Fortran versions, where O(1e-6) fractional agreement — not the ~0.2% emulation error — is the relevant tolerance.
- Your k values lie outside 9e-3 to 9 h/Mpc, where only the Eisenstein & Hu extrapolation (not the trained symbolic formula) applies.

## Inputs

- Cosmologies sampled from the LambdaCDM prior (e.g. Omega_m, h, sigma8/As, ns, Omega_b)
- k grid from 9e-3 to 9 h/Mpc at z=0
- Reference linear matter power spectra from CAMB for the same cosmologies and k grid
- Analytic emulated P(k) from the symbolic_pofk LambdaCDM linear emulator

## Outputs

- Comparison table (k, emulated P(k), CAMB P(k), fractional error) written to linear_pk_rms_error.csv
- Headline RMS fractional error over the full k range (~0.2% expected)

## How to apply

Load the packaged LambdaCDM linear emulator (examples/linear_example.py) and evaluate the analytic expression via plin_emulated()/logF_max_precision()/logF_fiducial(), keeping the default Eisenstein & Hu extrapolation (which includes baryons) outside the trained k range. Sample cosmologies spanning the LambdaCDM prior and evaluate the emulated linear P(k) with numpy on a grid of k from 9e-3 to 9 h/Mpc at z=0. Run CAMB for the same cosmologies and k grid to obtain the reference spectra, then compute pointwise fractional differences between the two and aggregate to the root-mean-squared fractional error over the full k range. Correctness requires the RMS fractional error to sit at the ~0.2% level reported in the paper's abstract, and the emulated-vs-reference differences must exceed the O(1e-6) Python-vs-Fortran implementation agreement so that emulation error, not implementation noise, dominates.

## Related tools

- **CAMB** (Boltzmann-equation reference solver; the analytic formula is reported as ~950 times faster to evaluate than CAMB)
- **numpy** (Evaluates the analytic linear P(k) on the k grid for sampled cosmologies)
- **symbolic_pofk** (Provides plin_emulated(), logF_max_precision(), logF_fiducial() and the linear_example.py script being validated) — github.com/DeaglanBartlett/symbolic_pofk

## Examples

```
git clone git@github.com:DeaglanBartlett/symbolic_pofk.git && cd symbolic_pofk && pip install . && python examples/linear_example.py
```

## Evaluation signals

- RMS fractional error between emulated and CAMB P(k) over k = 9e-3 to 9 h/Mpc is at the ~0.2% level reported in the abstract
- Pointwise fractional differences between the Python and Fortran implementations are O(1e-6), well below the emulation error, so emulation error dominates
- Emulated-vs-CAMB agreement is sub-percent across the full width of the sampled LambdaCDM prior
- Output CSV contains the expected columns (k, emulated P(k), CAMB P(k), fractional error) over the full k grid at z=0

## Limitations

- The ~0.2% accuracy claim holds only within k = 9e-3 to 9 h/Mpc; outside this range the emulator falls back to the Eisenstein & Hu fit (extrapolate=True by default), which is lower accuracy.
- The LambdaCDM linear emulator covers a fixed prior; wider priors or extended cosmologies (neutrino masses, w0, wa, baryonic effects) require the other emulators in the repository.
- Implementation-level agreement between the Python and Fortran codes is only O(1e-6), so benchmark harnesses must keep this noise below the ~0.2% emulation tolerance.

## Evidence

- [abstract] Symbolic emulator achieves 0.2% RMS fractional error vs CAMB over trained k range: [evidence span withheld — anchor status: not_found]
- [abstract] CAMB is the reference; the formula is ~950x faster: [evidence span withheld — anchor status: not_found]
- [methods] Eisenstein & Hu extrapolation is used outside the trained k range and can be disabled: "we use the Eisenstein & Hu fit which includes baryons. This can be switched off by setting extrapolate=False"
- [readme] Linear emulator example lives in examples/linear_example.py: "We give an example for how to use the ΛCDM
linear emulator in `examples/linear_example.py`."
- [readme] Python-vs-Fortran implementation agreement is O(1e-6), well below emulation error: "they are identical up to a fractional difference of
O(1e-6), which is much smaller than the error on the emulation"
