---
name: rime-ek-kernel-source-loop-extension
description: Use when adding a new source type (e.g. beta profiles, mentioned as future
  morphology) to Montblanc's EK kernel and you must verify GPU per-antenna terms and
  chi2 correctness.
when_to_use_negative:
- Your goal is to add direction-dependent gain (G_ps DIE / E_ps DDE matrix) evaluation
  — Montblanc operates on fully calibrated data and substitutes analytic expressions
  for E_ps terms
- You need multi-GPU or distributed computation — only single GPUs are currently supported
  and an out-of-core BIRO solution is not yet implemented
- Your source morphology already has an existing kernel loop (point or gaussian) —
  reuse those code paths instead of adding a new loop
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: Montblanc
  role: GPU RIME framework whose EK and B Sum kernels are extended with the new source
    loop
  repo: https://github.com/ska-sa/montblanc
- name: Python
  role: Implementation language for the kernel extension, providers, and test harness
- name: CUDA
  role: NVIDIA GPU architecture on which the Montblanc EK kernel source loop runs
- name: TensorFlow
  role: Backend version ('tf') used to configure the solver
- name: NumPy
  role: Supplies data source tiles and implements the independent CPU reference implementation
    for validation
  repo: https://github.com/numpy/numpy
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.003
    title: 'Montblanc : GPU accelerated radio interferometer measurement equations
      in support of Bayesian inference for radio observations'
  source_content_hash: sha256:f1338b5c4b9cf4da98bb413523a290ff34713d6a852873e21ce30d4dce2b3433
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ska-sa/montblanc
  tools:
  - CUDA
  - Montblanc
  - NumPy
  - Python
  - TensorFlow
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# rime-ek-kernel-source-loop-extension

## Summary

Method for extending Montblanc's GPU RIME pipeline with a new source morphology (e.g. beta-profile/King sources) by adding a source loop to the EK kernel and validating the resulting per-antenna terms and chi-squared against an independent CPU/numpy reference.

## When to use

Use when adding a new source type (e.g. beta profiles, mentioned as future morphology) to Montblanc's EK kernel and you must verify GPU per-antenna terms and chi2 correctness.

## When NOT to use

- Your goal is to add direction-dependent gain (G_ps DIE / E_ps DDE matrix) evaluation — Montblanc operates on fully calibrated data and substitutes analytic expressions for E_ps terms
- You need multi-GPU or distributed computation — only single GPUs are currently supported and an out-of-core BIRO solution is not yet implemented
- Your source morphology already has an existing kernel loop (point or gaussian) — reuse those code paths instead of adding a new loop

## Inputs

- Source model provider arrays as numpy tiles: lm coordinates, stokes parameters, spectral indices, and morphology shape parameters (e.g. beta values) sliced to dim_extents such as 'nsrc'/'ngsrc'
- Observed visibility data and weight vector for chi-squared computation
- Solver configuration: dtype ('double'), version ('tf'), and GPU memory budget (e.g. 4*1024*1024*1024)

## Outputs

- Per-antenna terms from the EK kernel for the new source morphology
- Model visibilities from the B Sum kernel reduction
- Scalar chi-squared value computed from model vs. observed visibilities
- A validation report (e.g. beta_profile_validation_report.md) recording max deviations vs. the CPU reference

## How to apply

Clone the Montblanc repository (git://github.com/ska-sa/montblanc.git) and install it in a virtual environment per the documented pip procedure. Implement the new source loop (e.g. beta-profile, King 1966/1972) inside the EK kernel so per-antenna terms are computed alongside the existing point and gaussian source loops, then write a data source provider (e.g. BetaSourceProvider) following the GaussianSourceProvider pattern — supplying lm coordinates, stokes parameters, spectral indices and beta shape parameters as numpy tiles sliced to the context's dim_extents and reporting source counts via updated_dimensions(). Configure the solver (montblanc.rime_solver_cfg(dtype='double', version='tf', mem_budget=4*1024*1024*1024)) and run slvr.solve with the provider registered in the source/sink provider lists. Implement an independent CPU/numpy (optionally numexpr-accelerated) reference computing the same per-antenna terms and the chi-squared — subtract model visibilities from observed data, square, multiply by weights, sum to a scalar — and compare GPU outputs across test extents within floating-point tolerance.

## Related tools

- **Montblanc** (GPU RIME framework whose EK and B Sum kernels are extended with the new source loop) — https://github.com/ska-sa/montblanc
- **Python** (Implementation language for the kernel extension, providers, and test harness)
- **CUDA** (NVIDIA GPU architecture on which the Montblanc EK kernel source loop runs)
- **TensorFlow** (Backend version ('tf') used to configure the solver)
- **NumPy** (Supplies data source tiles and implements the independent CPU reference implementation for validation) — https://github.com/numpy/numpy

## Examples

```
cfg = montblanc.rime_solver_cfg(dtype='double', version='tf', mem_budget=4*1024*1024*1024); slvr.solve(source_providers=[BetaSourceProvider(...)])
```

## Evaluation signals

- GPU per-antenna terms match the independent CPU/numpy reference within floating-point tolerance for all tested source/tile configurations
- GPU chi-squared values agree with the CPU reference chi2 computed as sum of weights times squared visibility residuals
- Provider tiles are correctly sliced to the context's dim_extents and source counts are reported through updated_dimensions()
- Max deviations across test extents are recorded in a validation report (beta_profile_validation_report.md)
- Solver runs with the double-precision 'tf' version configuration without dimension/schema registration errors

## Limitations

- Only single GPUs are currently supported; multi-GPU and clustered solutions are not yet available
- Problems exceeding the supplied GPU memory budget must be subdivided along the time dimension using multiple solvers with asynchronous transfers
- G_ps (DIE) matrix evaluation is not included as BIRO operates on fully calibrated data, and E_ps (DDE) terms are substituted with analytic expressions
- Beta profiles and other morphologies were architected for but not natively supported in the released version — extension requires the loop and provider pattern described here

## Evidence

- [discussion] per-antenna terms are computed in an EK kernel, and output to the GPU's main memory: [evidence span withheld — anchor status: not_found]
- [discussion] B Sum kernel combines per-antenna terms: [evidence span withheld — anchor status: not_found]
- [intro] chi-squared computation workflow: [evidence span withheld — anchor status: not_found]
- [discussion] beta profiles as supported future morphology: [evidence span withheld — anchor status: not_found]
