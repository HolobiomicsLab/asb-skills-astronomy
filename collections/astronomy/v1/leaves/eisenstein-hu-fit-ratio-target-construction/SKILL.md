---
name: eisenstein-hu-fit-ratio-target-construction
description: 'Use when building an interpretable symbolic emulator for P(k): instead
  of regressing the full spectrum directly, evaluate an existing low-accuracy fitting
  function for the same cosmologies and regress the fitting-function-to-CAMB ratio
  as the learning target.'
when_to_use_negative:
- You need the full non-linear power spectrum rather than the linear P(k) emulator
  covered by this ratio scheme
- You already have an accurate emulator (e.g. BACCO) and interpretability/speed of
  a closed-form analytic expression is not required
- The k range of interest lies far outside the trained range k = 9e-3 to 9 h/Mpc and
  the Eisenstein & Hu extrapolation is not acceptable accuracy
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_0102
tools:
- name: CAMB
  role: Solves the Boltzmann equations to produce the reference linear P(k) used as
    the denominator of the ratio targets
- name: Eisenstein & Hu fit
  role: Provides the low-accuracy analytic fitting function (including baryons) whose
    ratio to the CAMB solution forms the learning target; also used for extrapolation
    outside the trained k range
- name: Python
  role: Environment in which the emulators and ratio targets are constructed and exported
    as simple functions
  repo: github.com/deaglanbartlett/symbolic_pofk
- name: PyTorch
  role: Optional backend; install with pip install ".[torch]" to get the emulator
    as a torch function
  repo: github.com/deaglanbartlett/symbolic_pofk
- name: numpy
  role: Numerical library used in the results examples for working with the emulator
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202348811
    title: A precise symbolic emulator of the linear matter power spectrum
  source_content_hash: sha256:faaee592f9400cb3cb2c5e98b3123afacabd6cde9527af17d8830f685d619b2b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAMB
  - Eisenstein & Hu fit
  - PyTorch
  - Python
  - numpy
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# eisenstein-hu-fit-ratio-target-construction

## Summary

Constructs training targets for symbolic regression of the linear matter power spectrum by forming the ratio between the Eisenstein & Hu fitting function (including baryons) and the Boltzmann-equation solution from CAMB, so the learned expression inherits the physics that motivated the analytic approximation.

## When to use

Use when building an interpretable symbolic emulator for P(k): instead of regressing the full spectrum directly, evaluate an existing low-accuracy fitting function for the same cosmologies and regress the fitting-function-to-CAMB ratio as the learning target.

## When NOT to use

- You need the full non-linear power spectrum rather than the linear P(k) emulator covered by this ratio scheme
- You already have an accurate emulator (e.g. BACCO) and interpretability/speed of a closed-form analytic expression is not required
- The k range of interest lies far outside the trained range k = 9e-3 to 9 h/Mpc and the Eisenstein & Hu extrapolation is not acceptable accuracy

## Inputs

- CAMELS cosmological parameter grid (set of cosmologies)
- Boltzmann-equation linear P(k) solutions from CAMB for each cosmology
- Eisenstein & Hu fitting function (with baryons) evaluated at the same cosmologies and k values

## Outputs

- Ratio targets (Eisenstein & Hu P(k) divided by CAMB P(k)) for symbolic regression training
- Trained symbolic expression implementing the ratio, exportable as Python, torch, and/or Fortran 90 functions

## How to apply

First generate reference linear P(k) by solving the Boltzmann equations with CAMB for every cosmology in the CAMELS parameter grid. Then evaluate the Eisenstein & Hu fit which includes baryons for those same cosmologies, and form the ratio between the fitting function and the CAMB solution to serve as the regression target for the genetic-programming symbolic regression framework. Learning the ratio rather than P(k) itself means the final symbolic expression is the product of the trusted analytic approximation and a small correction function, which retains the physics that motivated the earlier approximation. Outside the trained k range (k = 9e-3 to 9 h/Mpc), the Eisenstein & Hu fit is reused for extrapolation, controlled by the extrapolate=True default in plin_emulated(), logF_max_precision() and logF_fiducial(); set extrapolate=False to disable. Validate by checking the trained expression reaches a root-mean-squared fractional error of 0.2% across the CAMELS cosmologies.

## Related tools

- **CAMB** (Solves the Boltzmann equations to produce the reference linear P(k) used as the denominator of the ratio targets)
- **Eisenstein & Hu fit** (Provides the low-accuracy analytic fitting function (including baryons) whose ratio to the CAMB solution forms the learning target; also used for extrapolation outside the trained k range)
- **Python** (Environment in which the emulators and ratio targets are constructed and exported as simple functions) — github.com/deaglanbartlett/symbolic_pofk
- **PyTorch** (Optional backend; install with pip install ".[torch]" to get the emulator as a torch function) — github.com/deaglanbartlett/symbolic_pofk
- **numpy** (Numerical library used in the results examples for working with the emulator)

## Examples

```
git clone git@github.com:DeaglanBartlett/symbolic_pofk.git; cd symbolic_pofk; pip install ".[torch]"
```

## Evaluation signals

- Trained expression attains a root-mean-squared fractional error of 0.2% between k = 9e-3 and 9 h/Mpc across the CAMELS cosmologies
- Symbolic emulator is 950 times faster to evaluate than CAMB and 36 times faster than BACCO
- With extrapolate=True (default), values outside the trained k range match the Eisenstein & Hu fit with baryons; setting extrapolate=False in plin_emulated(), logF_max_precision() and logF_fiducial() disables this
- Exported expression is a simple self-contained Python function that can be copied and modified to other languages

## Limitations

- Accuracy is validated only within k = 9e-3 to 9 h/Mpc across the CAMELS cosmologies; outside this range the code falls back to the Eisenstein & Hu fit, which has lower accuracy
- The emulator targets the redshift zero linear matter power spectrum in LCDM; extended cosmologies (neutrino masses, w0, wa), baryonic effects, and the non-linear spectrum are handled by separate emulators/papers
- Extrapolation via the Eisenstein & Hu fit can be switched off with extrapolate=False, in which case no extrapolation is provided

## Evidence

- [abstract] We learn the ratio between an existing low-accuracy fitting function forP(k) and that obtained by solving the Boltzmann equations: "We learn the ratio between an existing low-accuracy fitting function forP(k) and that obtained by solving the Boltzmann equations"
- [methods] we use the Eisenstein & Hu fit which includes baryons. This can be switched off by setting extrapolate=False: "we use the Eisenstein & Hu fit which includes baryons. This can be switched off by setting extrapolate=False"
- [methods] setting extrapolate=False in the functions plin_emulated(), logF_max_precision() and logF_fiducial(): "setting extrapolate=False in the functions plin_emulated(), logF_max_precision() and logF_fiducial()"
- [abstract] an analytic approximation to the linear power spectrum with a root mean squared fractional error of 0.2% between k = 9 × 10−3 − 9 h Mpc−1: [evidence span withheld — anchor status: not_found]
- [readme] by default, outside the k range tested in Bartlett et al. 2023, we use the Eisenstein & Hu fit which includes baryons: "by default, outside the k range tested in Bartlett et al. 2023, 
we use the Eisenstein & Hu fit which includes baryons"
