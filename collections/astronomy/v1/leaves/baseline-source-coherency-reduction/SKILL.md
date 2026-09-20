---
name: baseline-source-coherency-reduction
description: Use when reconstructing or re-implementing the RIME reduction pipeline
  in which per-antenna Jones terms (A[t,p,s,λ]) must be multiplied into per-baseline
  source coherencies and model visibilities, then reduced to a single χ² scalar for
  unit testing against a CPU numpy reference.
when_to_use_negative:
- Data are not fully calibrated — BIRO/Montblanc does not evaluate the Gps (DIE) matrices,
  so direction-independent gain calibration must be applied beforehand
- The goal is direction-dependent effect (DDE) modelling — Eps terms are substituted
  with analytic expressions, not full matrix evaluation
- Multi-GPU or out-of-core simulation is required — only single GPUs are supported
  and an out-of-core BIRO solution is not yet implemented
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Montblanc
  role: GPU (CUDA / TensorFlow) reference implementation of the RIME EK and B Sum
    kernels and χ² computation used as the validation target
  repo: https://github.com/ska-sa/montblanc
- name: NumPy
  role: CPU implementation of the three-kernel pipeline (EK, B Sum, χ² reduction)
    used as the numpy reference for unit testing
  repo: https://github.com/numpy/numpy
- name: CUDA
  role: Architecture on which the B Sum kernel reads per-antenna terms from GPU main
    memory and combines them into per-baseline terms
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.003
    title: 'Montblanc : GPU accelerated radio interferometer measurement equations
      in support of Bayesian inference for radio observations'
  source_content_hash: sha256:3ad929aaaefa4b91525988201c1cdd5591f62f13b79ba2469913f9dd183c9072
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ska-sa/montblanc
  tools:
  - CUDA
  - Montblanc
  - NumPy
  - Python
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# baseline-source-coherency-reduction

## Summary

Implements the second stage of Montblanc's three-kernel BIRO RIME workflow: the B Sum kernel, which reads per-antenna terms computed by the EK kernel, combines them into a 4D source-coherency array per baseline, reduces this to a 3D model visibility array, and feeds the χ² computation used by Bayesian inference.

## When to use

Use when reconstructing or re-implementing the RIME reduction pipeline in which per-antenna Jones terms (A[t,p,s,λ]) must be multiplied into per-baseline source coherencies and model visibilities, then reduced to a single χ² scalar for unit testing against a CPU numpy reference.

## When NOT to use

- Data are not fully calibrated — BIRO/Montblanc does not evaluate the Gps (DIE) matrices, so direction-independent gain calibration must be applied beforehand
- The goal is direction-dependent effect (DDE) modelling — Eps terms are substituted with analytic expressions, not full matrix evaluation
- Multi-GPU or out-of-core simulation is required — only single GPUs are supported and an out-of-core BIRO solution is not yet implemented

## Inputs

- Per-antenna terms array A[t,p,s,lambda] output from the EK kernel
- Hypercube-registered dimensions (ntime, antennas, channels, npsrc point sources, ngsrc Gaussian sources)
- Sky model data sources: lm coordinates, Stokes parameters, spectral indices (alpha), Gaussian shape parameters in single precision
- Synthetic observed visibilities and weight vector

## Outputs

- 4D per-baseline source-coherency array
- 3D model visibility array (time, baseline, channel)
- Single scalar χ² value comparable to the montblanc CPU reference (rime_solver_cfg with dtype='float', version='tf')

## How to apply

First run the EK (per-antenna) stage to produce per-antenna terms A[t,p,s,lambda] over time, antenna, source and channel dimensions (analytic expressions substituted for Eps DDE terms; Gps DIE matrices are not evaluated since BIRO assumes fully calibrated data). Then, in the B Sum stage, index the per-antenna terms by the two antennas of each baseline, form the 4D source-coherency array (2x2 Jones products per source), and sum over sources to reduce to a 3D model visibility array over time, baseline and channel. Register the problem through hypercube's register_dimension and register_array calls (e.g. 'npsrc' point and 'ngsrc' Gaussian sources) and serve lm coordinates, Stokes parameters, spectral indices and Gaussian shape parameters in single precision via data sources that slice tiles using context.dim_extents. Finally subtract model visibilities from synthetic observed visibilities, square, apply the weight vector, and sum to a single scalar χ², mirroring slvr.solve(source_providers=..., sink_providers=...).

## Related tools

- **Montblanc** (GPU (CUDA / TensorFlow) reference implementation of the RIME EK and B Sum kernels and χ² computation used as the validation target) — https://github.com/ska-sa/montblanc
- **NumPy** (CPU implementation of the three-kernel pipeline (EK, B Sum, χ² reduction) used as the numpy reference for unit testing) — https://github.com/numpy/numpy
- **CUDA** (Architecture on which the B Sum kernel reads per-antenna terms from GPU main memory and combines them into per-baseline terms)

## Examples

```
mb_solver = RimeSolver(rime_solver_cfg(dtype='float', version='tf')); chi2 = mb_solver.solve(source_providers=[...], sink_providers=[...])
```

## Evaluation signals

- The χ² scalar from the numpy three-kernel pipeline must match the montblanc CPU reference (rime_solver_cfg with dtype='float', version='tf') within single-precision floating-point tolerance
- Per-antenna terms A[t,p,s,lambda] must be indexed consistently by antenna pair for each baseline before coherency formation
- The 4D source-coherency array must reduce to a 3D model visibility array with dimensions (time, baseline, channel)
- The final χ² equals sum over visibilities of weight * |observed - model|^2, i.e. a single scalar after weighting
- Hypercube dimensions must be registered with the expected sizes (e.g. 10 'npsrc' and 10 'ngsrc' sources) and data sources must slice tiles via context.dim_extents

## Limitations

- Only single GPUs are currently supported; multi-GPU and clustered solutions are not yet available
- An out-of-core solution for BIRO is not yet implemented, and data-transfer bottlenecks for distributed BIRO remain unsolved
- G_ps (DIE) matrix evaluation is not included because BIRO operates on fully calibrated data
- Other source morphologies such as beta profiles are mentioned as future additions, not currently supported
- Problems larger than the GPU memory budget must be subdivided along the time dimension using multiple solvers with asynchronous transfers

## Evidence

- [intro] B Sum kernel combines per-antenna terms into per-baseline coherencies: [evidence span withheld — anchor status: not_found]
- [intro] χ² reduction to single scalar: [evidence span withheld — anchor status: not_found]
- [discussion] Kernel decomposition description: [evidence span withheld — anchor status: not_found]
- [intro] Time subdivision for memory budget: "if the problem is too large to fit into a supplied memory budget, the problem is subdivided along the time dimension."
- [methods] Hypercube dimension registration pattern: "cube.register_dimension("ntime", 10000, description="Timesteps")"
