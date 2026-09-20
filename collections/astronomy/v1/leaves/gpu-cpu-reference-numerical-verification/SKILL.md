---
name: gpu-cpu-reference-numerical-verification
description: Use when a new or modified GPU kernel in Montblanc (e.g. an EK-kernel
  source loop) must be proven numerically correct against an independent CPU reference
  before its per-antenna terms and chi-squared can be trusted for BIRO use.
when_to_use_negative:
- The task requires evaluation of direction-independent (G_ps) or full direction-dependent
  (E_ps) gain matrices, which Montblanc does not compute — it operates on fully calibrated
  data and substitutes analytic E_ps expressions
- 'The problem must run across multiple GPUs or a cluster: Montblanc currently supports
  only single GPUs, and the out-of-core/distributed solution is not implemented'
- The GPU kernel and CPU reference are not independent (e.g. the CPU code reuses the
  same kernel logic), which defeats the purpose of the cross-implementation check
tools:
- name: Montblanc
  role: GPU (TensorFlow/CUDA) RIME solver whose EK-kernel per-antenna terms and chi-squared
    are the outputs under verification; configured via montblanc.rime_solver_cfg
  repo: https://github.com/ska-sa/montblanc
- name: NumPy
  role: Independent CPU reference implementation used to recompute per-antenna terms,
    model visibilities and chi-squared for unit-test comparison
  repo: numpy/numpy
- name: numexpr
  role: Accelerates the numpy CPU reference computations used in the validation tests
- name: CUDA
  role: Underlying GPU architecture on which the EK and B Sum kernels run; the code
    under test
- name: TensorFlow
  role: Version backend for the Montblanc solver (version='tf') used when running
    slvr.solve for the verification
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.003
    title: 'Montblanc : GPU accelerated radio interferometer measurement equations
      in support of Bayesian inference for radio observations'
  source_content_hash: sha256:93c94fc6e89c123fefdc8bb11f5389bc71b13fc85ec2ecd3fefb0d26fddd15c7
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
  - numexpr
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# gpu-cpu-reference-numerical-verification

## Summary

Validates GPU-computed RIME outputs (per-antenna terms, model visibilities and chi-squared) by unit-testing them against an independent CPU reference implementation built with numpy and numexpr, as required when extending Montblanc's EK kernel with a new source morphology such as a beta-profile (King 1966/1972) loop.

## When to use

Use when a new or modified GPU kernel in Montblanc (e.g. an EK-kernel source loop) must be proven numerically correct against an independent CPU reference before its per-antenna terms and chi-squared can be trusted for BIRO use.

## When NOT to use

- The task requires evaluation of direction-independent (G_ps) or full direction-dependent (E_ps) gain matrices, which Montblanc does not compute — it operates on fully calibrated data and substitutes analytic E_ps expressions
- The problem must run across multiple GPUs or a cluster: Montblanc currently supports only single GPUs, and the out-of-core/distributed solution is not implemented
- The GPU kernel and CPU reference are not independent (e.g. the CPU code reuses the same kernel logic), which defeats the purpose of the cross-implementation check

## Inputs

- numpy-tiled source data: lm coordinates, stokes parameters, spectral indices (alpha), beta shape parameters, sliced to context dim_extents (nsrc/ngsrc-style)
- observed visibility data and weight vector
- solver configuration dictionary (montblanc.rime_solver_cfg with dtype='double', version='tf', mem_budget)
- independent CPU/numpy+numexpr reference implementation of the beta-profile RIME and chi-squared

## Outputs

- GPU-computed per-antenna terms from the EK kernel
- model visibility array (3D reduction of the 4D source-coherency array)
- single scalar chi-squared value
- validation report (beta_profile_validation_report.md) recording max deviations between GPU and CPU reference

## How to apply

First implement the new source loop (e.g. beta-profile) inside the EK kernel alongside the existing point and gaussian loops, and write a BetaSourceProvider following the GaussianSourceProvider pattern that supplies lm, stokes, spectral-index and beta shape parameters as numpy tiles sliced to the context's dim_extents (nsrc/ngsrc-style) and reported via updated_dimensions(). Next, implement an independent CPU reference in numpy (accelerated with numexpr) that recomputes the beta-profile per-antenna terms and chi-squared by subtracting model visibilities from the observed data, squaring, multiplying by the weight vector and summing to a single scalar. Configure the solver with montblanc.rime_solver_cfg(dtype='double', version='tf', mem_budget=4*1024*1024*1024) and run slvr.solve with the beta provider registered alongside the source/sink provider lists, using double precision to keep floating-point noise low for comparison. Compare GPU per-antenna terms and chi2 against the CPU reference across several test source/tile extents, record the maximum deviations in a validation report (e.g. beta_profile_validation_report.md), and pass only if agreement is within floating-point tolerance.

## Related tools

- **Montblanc** (GPU (TensorFlow/CUDA) RIME solver whose EK-kernel per-antenna terms and chi-squared are the outputs under verification; configured via montblanc.rime_solver_cfg) — https://github.com/ska-sa/montblanc
- **NumPy** (Independent CPU reference implementation used to recompute per-antenna terms, model visibilities and chi-squared for unit-test comparison) — numpy/numpy
- **numexpr** (Accelerates the numpy CPU reference computations used in the validation tests)
- **CUDA** (Underlying GPU architecture on which the EK and B Sum kernels run; the code under test)
- **TensorFlow** (Version backend for the Montblanc solver (version='tf') used when running slvr.solve for the verification)

## Examples

```
cfg = montblanc.rime_solver_cfg(dtype='double', version='tf', mem_budget=4*1024*1024*1024); slvr.solve(source_providers=[BetaSourceProvider(...)], sink_providers=[...])
```

## Evaluation signals

- GPU per-antenna terms match the independent numpy/numexpr CPU reference within floating-point tolerance for every tested source/tile configuration
- GPU chi-squared matches the CPU chi-squared (data minus model, squared, weight-multiplied, summed) within floating-point tolerance
- Maximum deviations across test extents are recorded in beta_profile_validation_report.md and are bounded by the chosen tolerance
- The beta provider's tiled data is correctly sliced to the context dim_extents (e.g. 'nsrc'/'ngsrc'-style extents) and source counts are reported via updated_dimensions()
- Solver runs in dtype='double' so residual differences are attributable to implementation, not single-precision noise

## Limitations

- Beta profiles are mentioned as future additions in the original paper — 'It is architected to allow addition of other source morphologies, such as beta profiles' — so no sanctioned beta implementation or reference exists upstream
- Only single GPUs are supported; multi-GPU and clustered RIME evaluation is not yet available, which limits test problem sizes to the GPU memory budget
- G_ps (DIE) matrices are not evaluated and E_ps (DDE) terms are analytic substitutions, so the verification covers only the terms Montblanc actually computes
- Data-transfer bottlenecks for distributed/out-of-core BIRO remain unsolved, capping how large validation workloads can be

## Evidence

- [intro] chi-squared definition used by both implementations: [evidence span withheld — anchor status: not_found]
- [discussion] EK kernel output under verification: [evidence span withheld — anchor status: not_found]
- [discussion] beta profiles as sanctioned extension point: [evidence span withheld — anchor status: not_found]
