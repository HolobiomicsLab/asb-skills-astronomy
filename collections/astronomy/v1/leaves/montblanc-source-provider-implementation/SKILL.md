---
name: montblanc-source-provider-implementation
description: 'Use when extending Montblanc''s RIME solver with a new source type:
  you must add a per-antenna-term kernel loop, a dimension-extent-driven data provider,
  and prove GPU/CPU numeric agreement for chi2.'
when_to_use_negative:
- 'The goal is calibration of direction-dependent effects: Montblanc operates on fully
  calibrated data and does not evaluate the G_ps (DIE) matrices'
- 'Data exceeds single-GPU memory with multi-GPU/distributed execution required: only
  single GPUs are currently supported'
- Validation against an existing sky-model morphology (point/gaussian) already covered
  by Montblanc's built-in providers
tools:
- name: Montblanc
  role: GPU RIME solver whose EK and B Sum kernels are extended with the new source
    loop and whose solver config registers the new provider
  repo: https://github.com/ska-sa/montblanc
- name: NumPy
  role: Supplies tiled source arrays for providers and the independent CPU reference
    implementation for validation
  repo: https://github.com/numpy/numpy
- name: CUDA
  role: Architecture on which Montblanc's dominant EK kernel (per-antenna terms) is
    implemented and extended
- name: TensorFlow
  role: Backend version ('tf') under which the solver is configured and run via montblanc.rime_solver_cfg
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.003
    title: 'Montblanc : GPU accelerated radio interferometer measurement equations
      in support of Bayesian inference for radio observations'
  source_content_hash: sha256:48afb999838e14879931407312d75d8f5c95f3a77c8a663dd49db27d0ab8fdf4
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# montblanc-source-provider-implementation

## Summary

Adds a new source morphology (e.g. beta-profile/King 1966/1972) to Montblanc by writing a new loop in the EK kernel plus a SourceProvider class following the GaussianSourceProvider pattern, and validates the GPU per-antenna terms and chi-squared against an independent numpy/numexpr CPU reference.

## When to use

Use when extending Montblanc's RIME solver with a new source type: you must add a per-antenna-term kernel loop, a dimension-extent-driven data provider, and prove GPU/CPU numeric agreement for chi2.

## When NOT to use

- The goal is calibration of direction-dependent effects: Montblanc operates on fully calibrated data and does not evaluate the G_ps (DIE) matrices
- Data exceeds single-GPU memory with multi-GPU/distributed execution required: only single GPUs are currently supported
- Validation against an existing sky-model morphology (point/gaussian) already covered by Montblanc's built-in providers

## Inputs

- numpy tiles of source lm coordinates sliced to context dim_extents (e.g. nsrc/ngsrc extents)
- Stokes parameters, spectral indices, and beta shape parameters for the new source type
- Observed visibilities and weight vector for chi2 comparison
- Solver configuration (dtype='double', version='tf', mem_budget e.g. 4*1024*1024*1024)

## Outputs

- GPU-computed per-antenna terms for beta-profile sources from the EK kernel
- Model visibilities and a single scalar chi2 value
- beta_profile_validation_report.md recording max GPU-vs-CPU deviations

## How to apply

Clone Montblanc (git://github.com/ska-sa/montblanc.git) and pip-install it in a virtual environment, then insert a beta-profile loop into the EK kernel so per-antenna terms for beta sources are computed alongside the existing point and gaussian loops (the codebase is architected to allow addition of other source morphologies, such as beta profiles). Write a BetaSourceProvider mirroring GaussianSourceProvider: methods slicing numpy tiles of lm coordinates, stokes parameters, spectral indices and beta shape parameters to the context's dim_extents ('nsrc'/'ngsrc'-style), with updated_dimensions() reporting the beta source count. Configure the solver with montblanc.rime_solver_cfg(dtype='double', version='tf', mem_budget=4*1024*1024*1024) and register the beta provider in the source-provider list of slvr.solve. Implement an independent CPU reference (numpy, optionally accelerated with numexpr) computing beta-profile per-antenna terms, then chi2 by subtracting model visibilities from observed data, squaring, multiplying by the weight vector and summing to a single scalar. Compare GPU vs CPU across multiple source/tile extents, recording max deviations in beta_profile_validation_report.md; agreement must be within floating-point tolerance.

## Related tools

- **Montblanc** (GPU RIME solver whose EK and B Sum kernels are extended with the new source loop and whose solver config registers the new provider) — https://github.com/ska-sa/montblanc
- **NumPy** (Supplies tiled source arrays for providers and the independent CPU reference implementation for validation) — https://github.com/numpy/numpy
- **CUDA** (Architecture on which Montblanc's dominant EK kernel (per-antenna terms) is implemented and extended)
- **TensorFlow** (Backend version ('tf') under which the solver is configured and run via montblanc.rime_solver_cfg)

## Examples

```
cfg = montblanc.rime_solver_cfg(dtype='double', version='tf', mem_budget=4*1024*1024*1024); slvr.solve(source_providers=[BetaSourceProvider(...)], sink_providers=[...])
```

## Evaluation signals

- GPU per-antenna terms for beta-profile sources match the numpy/numexpr CPU reference within floating-point tolerance across tested source/tile extents
- GPU chi2 (subtract model visibilities from observed data, square, multiply by weights, sum to scalar) equals the CPU reference chi2 within tolerance
- Provider slices return arrays shaped to the context's dim_extents (nsrc/ngsrc-style) without index errors for every tile
- updated_dimensions() on the BetaSourceProvider reports the correct beta source count reflected in solver dimensions
- Max deviations across all test configurations are recorded in beta_profile_validation_report.md

## Limitations

- Only single GPUs are supported; multi-GPU and clustered solutions are not yet available
- Beta profiles and other morphologies are future additions in the paper's scope, not built-in in the published version
- G_ps (DIE) matrices are not evaluated (fully calibrated data assumed) and E_ps (DDE) terms are substituted with analytic expressions
- Problems exceeding the GPU memory budget are subdivided along the time dimension using multiple solvers with asynchronous transfers

## Evidence

- [discussion] New source loop joins existing morphology loops: [evidence span withheld — anchor status: not_found]
- [discussion] EK kernel computes per-antenna terms the new loop extends: [evidence span withheld — anchor status: not_found]
- [intro] Chi2 reduction the CPU reference must reproduce: [evidence span withheld — anchor status: not_found]
