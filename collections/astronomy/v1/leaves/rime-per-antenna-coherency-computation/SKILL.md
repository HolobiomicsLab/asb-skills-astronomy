---
name: rime-per-antenna-coherency-computation
description: Use when simulating radio-interferometric visibilities from a sky model
  of point and Gaussian sources, or when a χ²/likelihood must be computed from model
  vs observed visibilities for Bayesian inference (BIRO).
when_to_use_negative:
- 'Data is not fully calibrated: evaluation of the direction-independent gain (G_ps
  DIE) matrices is not included in Montblanc, and E_ps (DDE) terms are substituted
  with analytic expressions rather than full matrix evaluation'
- 'Problem requires multi-GPU or clustered execution: only single GPUs are currently
  supported'
- Source morphology outside point or Gaussian shapes (e.g. beta profiles) is needed,
  as other morphologies are only future additions
tools:
- name: Montblanc
  role: GPU (CUDA/TensorFlow) implementation of the three-kernel RIME workflow (EK
    -> B Sum -> chi2 reduction) and CPU reference solver
  repo: https://github.com/ska-sa/montblanc
- name: Python
  role: Language for the numpy reference implementation of the three-kernel pipeline
    and data providers
- name: CUDA
  role: Architecture on which the EK and B Sum kernels are implemented
- name: NumPy
  role: numpy reference pipeline used to validate the χ² output against the montblanc
    CPU reference
  repo: https://github.com/numpy/numpy
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.003
    title: 'Montblanc : GPU accelerated radio interferometer measurement equations
      in support of Bayesian inference for radio observations'
  source_content_hash: sha256:b69de6ce5c40a3fb8ed51c96835c03a57f5828638b8d6e631e9224036fb91c9f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ska-sa/montblanc
  tools:
  - CUDA
  - Montblanc
  - NumPy
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# rime-per-antenna-coherency-computation

## Summary

Computes the Radio Interferometer Measurement Equation (RIME) by first evaluating expensive per-antenna terms A[t,p,s,λ] (EK kernel), combining them into per-baseline source coherencies and 3D model visibilities (B Sum kernel), and reducing to a single χ² scalar. This factored decomposition is what makes Montblanc's GPU-accelerated BIRO workflow tractable.

## When to use

Use when simulating radio-interferometric visibilities from a sky model of point and Gaussian sources, or when a χ²/likelihood must be computed from model vs observed visibilities for Bayesian inference (BIRO).

## When NOT to use

- Data is not fully calibrated: evaluation of the direction-independent gain (G_ps DIE) matrices is not included in Montblanc, and E_ps (DDE) terms are substituted with analytic expressions rather than full matrix evaluation
- Problem requires multi-GPU or clustered execution: only single GPUs are currently supported
- Source morphology outside point or Gaussian shapes (e.g. beta profiles) is needed, as other morphologies are only future additions

## Inputs

- Sky model source arrays in single precision: lm coordinates, Stokes parameters, spectral indices (point_alpha), Gaussian shape parameters
- Hypercube dimension/array registrations (e.g. cube.register_dimension('ntime', 10000, description='Timesteps'))
- Frequency channel data and uvw coordinates served via provider data sources (frequency, point_lm, point_stokes, gaussian_shape, etc.)
- Observed visibility data and weight vector for the χ² reduction

## Outputs

- Per-antenna terms array A[t,p,s,λ] (one complex Jones term per time, antenna, source, channel) from the EK stage
- 4D source-coherency array combined per baseline and reduced 3D model visibility array
- Single scalar χ² value suitable as a BIRO likelihood

## How to apply

Register the problem hypercube (dimensions such as ntime, and arrays) and serve source data (lm coordinates, Stokes parameters, spectral indices, Gaussian shapes) through provider-style data sources that slice tiles via context.dim_extents. First compute the EK (per-antenna) stage, producing per-antenna terms A[t,p,s,λ] for each time, antenna, source and channel using the paper's analytic expressions (Equations 3 and 4), where the analytic E_ps terms substitute for full direction-dependent-effect matrices. Then run the B Sum stage: read the per-antenna terms and combine them per baseline into a 4D source-coherency array, then reduce to a 3D model visibility array. Finally compute χ² by subtracting model visibilities from observed visibilities, squaring, multiplying by the weight vector, and summing to a single scalar; this χ² is the likelihood consumed by BIRO's MCMC sampler. Validate by reproducing the three-kernel pipeline in numpy and matching the montblanc CPU reference (rime_solver_cfg with dtype='float', version='tf') within single-precision tolerance.

## Related tools

- **Montblanc** (GPU (CUDA/TensorFlow) implementation of the three-kernel RIME workflow (EK -> B Sum -> chi2 reduction) and CPU reference solver) — https://github.com/ska-sa/montblanc
- **Python** (Language for the numpy reference implementation of the three-kernel pipeline and data providers)
- **CUDA** (Architecture on which the EK and B Sum kernels are implemented)
- **NumPy** (numpy reference pipeline used to validate the χ² output against the montblanc CPU reference) — https://github.com/numpy/numpy

## Examples

```
cube.register_dimension("ntime", 10000, description="Timesteps")  # then serve lm/Stokes providers and slvr.solve(source_providers=..., sink_providers=...)
```

## Evaluation signals

- The χ² scalar from the numpy three-kernel pipeline must match the montblanc CPU reference (dtype='float', version='tf') within single-precision floating-point tolerance
- EK-stage output must be a per-antenna array A[t,p,s,λ] indexed over time, antenna, source and channel, matching Equations 3 and 4
- B Sum stage must yield a 4D source-coherency array that reduces to a 3D model visibility array with correct baseline dimensionality
- The final reduction must produce a single scalar: subtract, square, weight-vector multiply, sum
- Data-source providers must return correctly sliced tiles matching context.dim_extents for each registered dimension

## Limitations

- G_ps (DIE) matrices are not evaluated since BIRO operates on fully calibrated data; E_ps (DDE) terms are substituted with analytic expressions
- Only single GPUs are supported; multi-GPU and clustered solutions are not yet available, and an out-of-core solution for BIRO with its data-transfer bottlenecks remains unimplemented
- Problems exceeding the GPU memory budget must be subdivided along the time dimension using multiple solvers with asynchronous transfers
- Other source morphologies such as beta profiles are not currently supported

## Evidence

- [intro] Per-antenna terms combined into coherencies: [evidence span withheld — anchor status: not_found]
- [discussion] EK kernel outputs per-antenna terms to GPU memory: [evidence span withheld — anchor status: not_found]
- [discussion] B Sum kernel combines per-antenna terms per baseline: [evidence span withheld — anchor status: not_found]
- [intro] chi2 reduction definition: [evidence span withheld — anchor status: not_found]
