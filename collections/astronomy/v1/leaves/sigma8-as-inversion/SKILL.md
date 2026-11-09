---
name: sigma8-as-inversion
description: Use when an analysis needs fast sigma8 <-> As conversion across a sampled
  LambdaCDM cosmology grid, or when verifying the published 0.1% accuracy of the analytic
  sigma8 expression against CAMB.
when_to_use_negative:
- The task needs the linear/nonlinear matter power spectrum P(k) itself rather than
  the sigma8 <-> As scalar conversion — use the P(k) emulator skills instead
- Cosmologies outside the published prior range of the Bartlett et al. 2023 linear
  emulator (e.g. requiring massive neutrinos, w0/wa, or baryonic effects) — use the
  SYREN-NEW or baryonic emulators cited in the README
- Sub-0.1% accuracy in sigma8 is required — the analytic expression's ~0.1% RMS error
  is the model-level floor
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: symbolic_pofk
  role: Provides the analytic sigma8(As) expression and the linear.sigma8_to_As /
    As_to_sigma8 conversion functions used for forward evaluation and inversion
  repo: github.com/DeaglanBartlett/symbolic_pofk
- name: CAMB
  role: Boltzmann-equation solver used to compute reference sigma8 values against
    which the analytic expression is validated
- name: Python
  role: Language in which the emulators are given as simple functions; used to implement
    the root solver for inversion
- name: numpy
  role: Used to evaluate the analytic expression and implement Newton/bisection root
    solving for the A_s inversion
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202348811
    title: A precise symbolic emulator of the linear matter power spectrum
  source_content_hash: sha256:7167d16718eccf7fa2768563d5b494d81c8bd2b12a074b38a9b8709420943c97
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAMB
  - Python
  - numpy
  - symbolic_pofk
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# sigma8-as-inversion

## Summary

Validate and invert the symbolic_pofk analytic expression for converting A_s to sigma8, which reaches ~0.1% RMS fractional error against CAMB and can be numerically inverted to recover A_s from sigma8.

## When to use

Use when an analysis needs fast sigma8 <-> As conversion across a sampled LambdaCDM cosmology grid, or when verifying the published 0.1% accuracy of the analytic sigma8 expression against CAMB.

## When NOT to use

- The task needs the linear/nonlinear matter power spectrum P(k) itself rather than the sigma8 <-> As scalar conversion — use the P(k) emulator skills instead
- Cosmologies outside the published prior range of the Bartlett et al. 2023 linear emulator (e.g. requiring massive neutrinos, w0/wa, or baryonic effects) — use the SYREN-NEW or baryonic emulators cited in the README
- Sub-0.1% accuracy in sigma8 is required — the analytic expression's ~0.1% RMS error is the model-level floor

## Inputs

- Grid of LambdaCDM cosmological parameters (e.g. Omega_m, ns, sigma8 or A_s) sampled across the Bartlett et al. 2023 linear-emulator prior range
- Reference sigma8 values from CAMB runs for each sampled cosmology
- symbolic_pofk analytic sigma8(As) conversion function (Python, numpy)

## Outputs

- Per-cosmology table of fractional errors and RMS/maximum summaries (sigma8_rms_error.csv)
- Per-cosmology sigma8 -> As -> sigma8 round-trip fractional errors with RMS and maximum summaries (sigma8_as_roundtrip_errors.csv)
- Recovered A_s values for each sampled cosmology

## How to apply

Install the symbolic_pofk package (git clone github.com/DeaglanBartlett/symbolic_pofk; pip install .) and import the sigma8 <-> As conversion function following the parameter conventions of examples/linear_example.py. Sample LambdaCDM cosmologies (e.g. Omega_m, ns, sigma8) across the same prior range used to train the linear P(k) emulator of Bartlett et al. 2023 (arXiv:2311.15865). For validation, evaluate the analytic sigma8(As) expression with numpy, run CAMB for each cosmology to obtain reference sigma8 from the Boltzmann-equation solution, and compute per-cosmology fractional errors (analytic - CAMB)/CAMB, aggregating to an RMS; the published target is a 0.1% RMS fractional error. For inversion, recover A_s from sigma8 using a numpy-implemented Newton or bisection root solve with tolerance far tighter than the 0.1% model-level error, then re-evaluate the forward conversion at the recovered A_s to record round-trip fractional differences. Write per-cosmology tables plus RMS/maximum summaries (e.g. sigma8_rms_error.csv, sigma8_as_roundtrip_errors.csv).

## Related tools

- **symbolic_pofk** (Provides the analytic sigma8(As) expression and the linear.sigma8_to_As / As_to_sigma8 conversion functions used for forward evaluation and inversion) — github.com/DeaglanBartlett/symbolic_pofk
- **CAMB** (Boltzmann-equation solver used to compute reference sigma8 values against which the analytic expression is validated)
- **Python** (Language in which the emulators are given as simple functions; used to implement the root solver for inversion)
- **numpy** (Used to evaluate the analytic expression and implement Newton/bisection root solving for the A_s inversion)

## Examples

```
git clone git@github.com:DeaglanBartlett/symbolic_pofk.git; cd symbolic_pofk; pip install .; python -c "from symbolic_pofk.linear import As_to_sigma8, sigma8_to_As"
```

## Evaluation signals

- RMS fractional error of analytic sigma8 vs CAMB stays at the ~0.1% level across the same cosmology range as the P(k) emulator
- RMS round-trip fractional error of sigma8 -> As -> sigma8 remains at the ~0.1% level across the sampled cosmologies
- Pure numerical inversion residuals are no larger than O(1e-6), much smaller than the error on the emulation
- Root-solver tolerance on A_s is far tighter than the 0.1% model-level error of the conversion formula
- Per-cosmology tables include both the fractional error per cosmology and aggregate RMS/maximum summaries

## Limitations

- The 0.1% RMS fractional accuracy of the analytic sigma8 expression is only established across the same range of cosmologies used to train the linear P(k) emulator of Bartlett et al. 2023
- Inversion via root solving is exact only up to numerical tolerance; pure numerical residuals (O(1e-6) in the Python/Fortran comparison) must be kept far below the 0.1% model-level error to be negligible

## Evidence

- [abstract] Analytic approximation for sigma8 has 0.1% RMS error and is invertible: [evidence span withheld — anchor status: not_found]
- [intro] Conversion between sigma8 and As is provided as a function of cosmology: "Conversion $\sigma_8 \leftrightarrow A_{\rm s}$
as a function of cosmology."
- [readme] Emulators are installed via git clone and pip install: "To install the emulators and the dependencies, run the following
git clone ... cd symbolic_pofk, pip install ."
- [methods] Linear emulator usage example defines parameter conventions: "We give an example for how to use the ΛCDM
linear emulator in `examples/linear_example.py`."
- [readme] Python vs Fortran implementations agree to O(1e-6): "they are identical up to a fractional difference of
O(1e-6), which is much smaller than the error on the emulation"
