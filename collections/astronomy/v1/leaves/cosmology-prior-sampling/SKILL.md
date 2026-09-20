---
name: cosmology-prior-sampling
description: Use when an MCMC, grid, or validation study needs LambdaCDM parameter
  vectors sampled across the paper's prior range to evaluate the syren linear P(k)
  emulator or the invertible sigma8 <-> As conversion at z=0.
when_to_use_negative:
- The cosmology includes massive neutrinos, w0, or wa — use the extended SYREN-NEW
  emulators (examples/syren_new_example.ipynb) instead of the LambdaCDM linear emulator
- The k range of interest lies outside 9e-3 to 9 h/Mpc and you need trained-range
  precision rather than the Eisenstein & Hu extrapolation
- Input is already a validated As value for a fixed cosmology, so no prior sampling
  or inversion is needed
tools:
- name: symbolic_pofk
  role: Package providing the analytic linear P(k) emulator and the invertible sigma8
    <-> As conversion evaluated on sampled cosmologies
  repo: https://github.com/DeaglanBartlett/symbolic_pofk
- name: numpy
  role: Vectorized evaluation of the analytic expressions on the cosmology sample
    and k grid; root solver for inverting sigma8(As)
- name: CAMB
  role: Boltzmann-equation reference spectra used to validate the RMS fractional accuracy
    of predictions on the sampled cosmologies
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202348811
    title: A precise symbolic emulator of the linear matter power spectrum
  source_content_hash: sha256:7ced4b4597760f59fbfbbb9c772e382c35f443a74de424ffcb5244007d02d939
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DeaglanBartlett/symbolic_pofk
  tools:
  - CAMB
  - numpy
  - symbolic_pofk
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cosmology-prior-sampling

## Summary

Sample a grid of LambdaCDM cosmologies (e.g. Omega_m, ns, sigma8) across the published prior range and push them through the syren symbolic emulators — the analytic sigma8(As) expression and the linear P(k) formula — to generate parameter-locked predictions with quantified round-trip and RMS fractional errors.

## When to use

Use when an MCMC, grid, or validation study needs LambdaCDM parameter vectors sampled across the paper's prior range to evaluate the syren linear P(k) emulator or the invertible sigma8 <-> As conversion at z=0.

## When NOT to use

- The cosmology includes massive neutrinos, w0, or wa — use the extended SYREN-NEW emulators (examples/syren_new_example.ipynb) instead of the LambdaCDM linear emulator
- The k range of interest lies outside 9e-3 to 9 h/Mpc and you need trained-range precision rather than the Eisenstein & Hu extrapolation
- Input is already a validated As value for a fixed cosmology, so no prior sampling or inversion is needed

## Inputs

- Grid or random samples of LambdaCDM cosmological parameters (e.g. Omega_m, ns, sigma8 or As) spanning the published prior range
- k grid from 9e-3 to 9 h/Mpc at z=0 for P(k) evaluation
- symbolic_pofk package functions (plin_emulated(), logF_max_precision(), logF_fiducial(), sigma8 <-> As conversion)

## Outputs

- Per-cosmology predictions: emulated linear P(k) values or recovered As from sigma8
- sigma8_as_roundtrip_errors.csv with per-cosmology fractional round-trip errors plus RMS and maximum summary values
- linear_pk_rms_error.csv with k, emulated P(k), CAMB P(k), and fractional error columns

## How to apply

Install the emulators from the cloned repository (git clone git@github.com:DeaglanBartlett/symbolic_pofk.git; cd symbolic_pofk; pip install .) and follow the parameter conventions of examples/linear_example.py. Draw cosmologies spanning the LambdaCDM prior (e.g. Omega_m, ns, sigma8/As) and evaluate the analytic expressions with numpy on a k grid from 9e-3 to 9 h/Mpc at z=0, keeping the default Eisenstein & Hu extrapolation (which includes baryons) outside the trained k range. When sigma8 is the sampling variable, invert the analytic sigma8(As) formula numerically — Newton iteration or bisection on As with a solver tolerance far tighter than the 0.1% model-level error — to recover As for each cosmology. Re-evaluate the forward conversion at the recovered As and record fractional round-trip differences, plus RMS and maximum summary values. For emulator-vs-CAMB comparisons, compute pointwise fractional differences and aggregate to the RMS fractional error over the full k range.

## Related tools

- **symbolic_pofk** (Package providing the analytic linear P(k) emulator and the invertible sigma8 <-> As conversion evaluated on sampled cosmologies) — https://github.com/DeaglanBartlett/symbolic_pofk
- **numpy** (Vectorized evaluation of the analytic expressions on the cosmology sample and k grid; root solver for inverting sigma8(As))
- **CAMB** (Boltzmann-equation reference spectra used to validate the RMS fractional accuracy of predictions on the sampled cosmologies)

## Examples

```
git clone git@github.com:DeaglanBartlett/symbolic_pofk.git; cd symbolic_pofk; pip install .; python examples/linear_example.py
```

## Evaluation signals

- RMS fractional error of emulated linear P(k) vs CAMB over k = 9e-3 to 9 h/Mpc stays at the ~0.2% level reported in the abstract
- RMS round-trip sigma8 -> As -> sigma8 fractional error remains at the ~0.1% level of the conversion formula across the cosmology range
- Pure numerical residuals from inversion and implementation are no larger than O(1e-6), much smaller than the emulation error
- Extrapolation behaviour matches the Eisenstein & Hu fit (including baryons) outside the trained k range, unless extrapolate=False is set deliberately
- Recovered As values stay within the published prior range and the root solver converges to a tolerance far tighter than 0.1%

## Limitations

- The LambdaCDM emulators are only trained within the published prior range; outside the k range the default Eisenstein & Hu extrapolation is used and can be switched off with extrapolate=False
- The Python and Fortran implementations agree only to a fractional difference of O(1e-6), which must be kept below the emulation error when cross-checking
- The sigma8(As) inversion inherits the 0.1% RMS fractional error of the analytic sigma8 approximation, so recovered As carries that model-level uncertainty

## Evidence

- [abstract] abstract: [evidence span withheld — anchor status: not_found]
- [methods] methods: "We give an example for how to use the ΛCDM linear emulator in examples/linear_example.py."
- [readme] readme-install: "To install the emulators and the dependencies, run the following"
- [readme] readme-fortran: "they are identical up to a fractional difference of
O(1e-6), which is much smaller than the error on the emulation)"
- [abstract] abstract-accuracy: [evidence span withheld — anchor status: not_found]
