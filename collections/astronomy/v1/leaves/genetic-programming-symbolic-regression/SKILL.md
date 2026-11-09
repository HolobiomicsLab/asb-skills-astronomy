---
name: genetic-programming-symbolic-regression
description: Use when a fast, interpretable analytic replacement for an expensive
  numerical solver (e.g. Boltzmann-code power spectra) is needed, and an existing
  low-accuracy fitting function exists whose ratio to the ground truth can be regressed.
when_to_use_negative:
- The target function has no existing low-accuracy analytic approximation whose ratio
  to the ground truth can be formed — the ratio-learning strategy depends on this
  anchor
- Precision requirements exceed 0.2% RMS fractional error over k = 9×10⁻³ – 9 h Mpc⁻¹,
  or cosmologies fall outside the trained CAMELS parameter grid without accepting
  Eisenstein & Hu extrapolation
- A non-interpretable but more accurate emulator is acceptable — a neural-network
  emulator such as BACCO may be preferred despite being ~36× slower
edam_operation: http://edamontology.org/operation_3946
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: CAMB
  role: Solves the Boltzmann equations to produce the reference linear P(k) targets
    for every CAMELS cosmology
- name: Eisenstein & Hu fit
  role: Low-accuracy analytic fitting function (with baryons) whose ratio to the CAMB
    solution is regressed; also used for extrapolation outside the trained k range
- name: Python
  role: Language in which the emulators are delivered as simple copy-paste functions
    and the package is installed via pip
  repo: github.com/DeaglanBartlett/symbolic_pofk
- name: PyTorch
  role: Optional backend, installed with pip install ".[torch]"; some emulators are
    also given as torch functions
  repo: github.com/DeaglanBartlett/symbolic_pofk
- name: Fortran 90
  role: Optional compiled implementation produced by ./setup_fortran.sh, identical
    to the Python version up to a fractional difference of O(1e-6)
  repo: github.com/DeaglanBartlett/symbolic_pofk
- name: numpy
  role: Numerical library used in the example scripts for evaluating and plotting
    the emulator
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202348811
    title: A precise symbolic emulator of the linear matter power spectrum
  source_content_hash: sha256:5e5b6378f35471e1b9e194dd7fd8f3f7ff807bb0604b9109c8a6ebb451ab8551
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAMB
  - Eisenstein & Hu fit
  - Fortran 90
  - PyTorch
  - Python
  - numpy
  edam_operation: http://edamontology.org/operation_3946
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# genetic-programming-symbolic-regression

## Summary

A genetic-programming based symbolic regression workflow that learns compact analytic expressions for the ratio between the Eisenstein & Hu P(k) fitting function (with baryons) and the CAMB Boltzmann-equation solution, yielding an interpretable emulator of the linear matter power spectrum accurate to 0.2% RMS fractional error.

## When to use

Use when a fast, interpretable analytic replacement for an expensive numerical solver (e.g. Boltzmann-code power spectra) is needed, and an existing low-accuracy fitting function exists whose ratio to the ground truth can be regressed.

## When NOT to use

- The target function has no existing low-accuracy analytic approximation whose ratio to the ground truth can be formed — the ratio-learning strategy depends on this anchor
- Precision requirements exceed 0.2% RMS fractional error over k = 9×10⁻³ – 9 h Mpc⁻¹, or cosmologies fall outside the trained CAMELS parameter grid without accepting Eisenstein & Hu extrapolation
- A non-interpretable but more accurate emulator is acceptable — a neural-network emulator such as BACCO may be preferred despite being ~36× slower

## Inputs

- CAMELS cosmology parameter grid
- CAMB Boltzmann-equation solutions for the linear matter power spectrum P(k)
- Eisenstein & Hu fitting function (including baryons) evaluated at the same cosmologies
- Ratio targets between the fitting function and the CAMB solution

## Outputs

- Interpretable symbolic expression for the P(k) ratio, exported as a Python function
- Optionally torch and/or Fortran 90 implementations of the same expression
- Validation metric: root-mean-squared fractional error (target 0.2% for k = 9×10⁻³ – 9 h Mpc⁻¹)

## How to apply

Clone the symbolic_pofk repository and install it with `pip install .` (or `pip install ".[torch]"` for the PyTorch backend; `./setup_fortran.sh` compiles a Fortran 90 version with a Python wrapper). Generate reference linear P(k) by solving the Boltzmann equations with CAMB for every cosmology in the CAMELS parameter grid, evaluate the Eisenstein & Hu fitting function including baryons for the same cosmologies, and form ratio targets between the fitting function and the CAMB solution — regressing the ratio rather than P(k) itself is what transfers the physics already encoded in the analytic approximation. Run the genetic-programming symbolic regression framework over candidate mathematical expressions, then export the best expression as a simple Python function (optionally also torch/Fortran 90). Validate that the trained expression attains a root-mean-squared fractional error of 0.2% between k = 9×10⁻³ and 9 h Mpc⁻¹ across the CAMELS cosmologies; outside the trained k range, extrapolate with the Eisenstein & Hu fit with baryons (extrapolate=True by default, set extrapolate=False to disable).

## Related tools

- **CAMB** (Solves the Boltzmann equations to produce the reference linear P(k) targets for every CAMELS cosmology)
- **Eisenstein & Hu fit** (Low-accuracy analytic fitting function (with baryons) whose ratio to the CAMB solution is regressed; also used for extrapolation outside the trained k range)
- **Python** (Language in which the emulators are delivered as simple copy-paste functions and the package is installed via pip) — github.com/DeaglanBartlett/symbolic_pofk
- **PyTorch** (Optional backend, installed with pip install ".[torch]"; some emulators are also given as torch functions) — github.com/DeaglanBartlett/symbolic_pofk
- **Fortran 90** (Optional compiled implementation produced by ./setup_fortran.sh, identical to the Python version up to a fractional difference of O(1e-6)) — github.com/DeaglanBartlett/symbolic_pofk
- **numpy** (Numerical library used in the example scripts for evaluating and plotting the emulator)

## Examples

```
git clone git@github.com:DeaglanBartlett/symbolic_pofk.git && cd symbolic_pofk && pip install ".[torch]"; python examples/linear_example.py
```

## Evaluation signals

- Trained expression attains a root-mean-squared fractional error of 0.2% between k = 9×10⁻³ and 9 h Mpc⁻¹ across the CAMELS cosmologies
- Emulator evaluation is ~950× faster than CAMB and ~36× faster than the BACCO neural-network emulator
- Fortran and Python implementations agree to a fractional difference of O(1e-6), far smaller than the emulation error
- Disabling extrapolation (extrapolate=False in plin_emulated(), logF_max_precision(), logF_fiducial()) changes behavior only outside the trained k range
- The exported expression is a simple, copyable Python function reproducible from the repository examples (e.g. examples/linear_example.py)

## Limitations

- Accuracy is validated only over k = 9×10⁻³ – 9 h Mpc⁻¹ and across the CAMELS cosmology grid; outside this range the Eisenstein & Hu fit with baryons is used by default (extrapolate=True)
- The symbolic emulator trades some precision relative to Boltzmann solvers in exchange for interpretability and ~950× speed-up over CAMB
- Example results are for redshift zero; extension to massive neutrinos, w0, wa, baryonic effects, or wider priors requires the companion emulators/papers

## Evidence

- [abstract] ratio-learning strategy: "We learn the ratio between an existing low-accuracy fitting function forP(k) and that obtained by solving the Boltzmann equations"
- [abstract] genetic programming search: "We utilise an efficient genetic programming based symbolic regression framework to explore the space of potential mathematical expressions"
- [abstract] accuracy and speed: [evidence span withheld — anchor status: not_found]
- [methods] installation: "If you wish to use the pytorch code, the third line should be replaced with pip install ".[torch]""
- [readme] extrapolation behavior: "we use the Eisenstein & Hu fit which includes baryons. This can be switched off by setting
`extrapolate=False` in the functions `plin_emulated()`, `logF_max_precision()` and
`logF_fiducial()`"
- [readme] fortran wrapper: "they are identical up to a fractional difference of
O(1e-6), which is much smaller than the error on the emulation"
