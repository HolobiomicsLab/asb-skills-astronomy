---
name: sigma8-emulator-accuracy-benchmarking
description: Use when you need a fast, closed-form σ8 ↔ A_s conversion for a sampled
  ΛCDM cosmology and must first verify that the analytic expression matches CAMB to
  ~0.1% RMS fractional error over the same parameter domain as the Bartlett et al.
  2023 P(k) emulator.
when_to_use_negative:
- When the task concerns the P(k) emulator's 0.2% RMS fractional accuracy over k =
  9×10⁻³ − 9 h Mpc⁻¹ rather than the σ8 ↔ A_s conversion
- When cosmologies lie outside the ΛCDM range of the linear emulator (e.g. massive
  neutrinos, w0–wa) — use the syren-new extended-cosmology emulators instead
- When a full Boltzmann solve is already affordable and exactness matters more than
  the ~950× speed-up over CAMB
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: symbolic_pofk
  role: Source of the analytic σ8 expression and Python emulator functions evaluated
    at each sampled cosmology
  repo: github.com/deaglanbartlett/symbolic_pofk
- name: CAMB
  role: Boltzmann-equation reference solver providing the ground-truth σ8 per cosmology
- name: numpy
  role: Vectorized evaluation of the analytic expression and RMS fractional-error
    aggregation
- name: Python
  role: Language in which the emulators are given as simple functions that can be
    copied, pasted and modified
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202348811
    title: A precise symbolic emulator of the linear matter power spectrum
  source_content_hash: sha256:75a7c555078def9b8c05013133b9d9fa152284a797752854ad87edc7e650e4ad
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAMB
  - Python
  - numpy
  - symbolic_pofk
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# sigma8-emulator-accuracy-benchmarking

## Summary

Benchmarks the symbolic_pofk analytic sigma8 expression (from Bartlett et al. 2024, A&A 686 A209) against Boltzmann-equation sigma8 from CAMB across the ΛCDM cosmology range used to train the linear P(k) emulator, verifying the published 0.1% RMS fractional accuracy and its invertibility for A_s.

## When to use

Use when you need a fast, closed-form σ8 ↔ A_s conversion for a sampled ΛCDM cosmology and must first verify that the analytic expression matches CAMB to ~0.1% RMS fractional error over the same parameter domain as the Bartlett et al. 2023 P(k) emulator.

## When NOT to use

- When the task concerns the P(k) emulator's 0.2% RMS fractional accuracy over k = 9×10⁻³ − 9 h Mpc⁻¹ rather than the σ8 ↔ A_s conversion
- When cosmologies lie outside the ΛCDM range of the linear emulator (e.g. massive neutrinos, w0–wa) — use the syren-new extended-cosmology emulators instead
- When a full Boltzmann solve is already affordable and exactness matters more than the ~950× speed-up over CAMB

## Inputs

- Sampled ΛCDM cosmological parameter sets covering the Bartlett et al. 2023 (arXiv:2311.15865) linear-emulator cosmology domain
- symbolic_pofk analytic σ8 expression (Python emulator functions)
- CAMB reference σ8 values for the same cosmologies

## Outputs

- Per-cosmology table of fractional errors written to sigma8_rms_error.csv
- RMS fractional error summary (target ≈ 0.1%)
- Inverted A_s(σ8, other cosmological parameters) values from the analytic function

## How to apply

Clone github.com/DeaglanBartlett/symbolic_pofk and install with 'pip install .'. Sample ΛCDM cosmological parameters over the same domain used to train the linear matter power spectrum emulator of Bartlett et al. 2023 (arXiv:2311.15865). For each sampled cosmology, evaluate the analytic σ8 formula via the Python emulator functions with numpy, then run CAMB on the identical cosmology to obtain reference σ8 from the Boltzmann-equation solution. Compute the per-cosmology fractional error (analytic σ8 − CAMB σ8)/CAMB σ8 and aggregate to a root-mean-squared fractional error; validate that the RMS reproduces the published accuracy of 0.1%. The same analytic function is easily invertible to recover A_s as a function of σ8 and the other cosmological parameters, which can be checked by round-tripping sampled (A_s, σ8) pairs.

## Related tools

- **symbolic_pofk** (Source of the analytic σ8 expression and Python emulator functions evaluated at each sampled cosmology) — github.com/deaglanbartlett/symbolic_pofk
- **CAMB** (Boltzmann-equation reference solver providing the ground-truth σ8 per cosmology)
- **numpy** (Vectorized evaluation of the analytic expression and RMS fractional-error aggregation)
- **Python** (Language in which the emulators are given as simple functions that can be copied, pasted and modified)

## Examples

```
git clone git@github.com:DeaglanBartlett/symbolic_pofk.git; cd symbolic_pofk; pip install .
```

## Evaluation signals

- RMS fractional error of analytic σ8 vs CAMB σ8 across the sampled cosmology range reproduces the published level of 0.1%
- Per-cosmology fractional errors (analytic σ8 − CAMB σ8)/CAMB σ8 are computed and persisted in sigma8_rms_error.csv
- Inversion check: analytic function inverted to obtain A_s as a function of σ8 and other cosmological parameters recovers the input A_s values
- Sampled cosmologies lie within the same cosmology range used to train the Bartlett et al. 2023 linear P(k) emulator
- Emulator was installed per README ('git clone ... pip install .') so the exact published expression is exercised

## Limitations

- The 0.1% accuracy holds only across the cosmology range of the Bartlett et al. 2023 linear emulator; outside it the analytic expression is unvalidated
- For ΛCDM linear/nonlinear emulators, outside the tested k range the code falls back to the Eisenstein & Hu fit (includes baryons), controllable via extrapolate=False in plin_emulated(), logF_max_precision() and logF_fiducial()
- Python and Fortran implementations agree only up to a fractional difference of O(1e-6), which is much smaller than the emulation error but nonzero

## Evidence

- [abstract] Analytic approximation for σ8 has a root mean squared fractional error of 0.1% across the same range of cosmologies and is invertible to obtain As: [evidence span withheld — anchor status: not_found]
- [readme] Workflow installs the package via pip and evaluates with the Python emulator functions: "git clone git@github.com:DeaglanBartlett/symbolic_pofk.git
cd symbolic_pofk
pip install ."
- [intro] Conversion sigma_8 <-> A_s is provided as a function of cosmology: "* Conversion $\sigma_8 \leftrightarrow A_{\rm s}$
as a function of cosmology."
- [intro] Emulators are given as simple Python functions: "Here we give the emulators as simple python functions."
