---
name: rime-chi-squared-visibility-computation
description: Reference for computing model visibilities and a single chi-squared value
  from a radio interferometry sky model (point/gaussian/beta sources) via the RIME,
  especially when unit-testing a GPU kernel against a numpy CPU reference or feeding
  likelihoods to Bayesian sampling.
when_to_use_negative:
- 'Data is not fully calibrated: Montblanc does not evaluate G_ps (DIE) matrices,
  so direction-independent gain errors must be solved elsewhere first'
- The analysis requires imaging, deconvolution or source extraction rather than visibility-domain
  chi-squared likelihood evaluation
- 'Multi-GPU or distributed execution is required: only single GPUs are supported
  and the out-of-core BIRO solution with its data-transfer bottlenecks is not yet
  implemented'
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: Montblanc
  role: GPU (CUDA/TensorFlow) RIME solver producing per-antenna terms, model visibilities
    and chi-squared; invoked via slvr.solve with source and sink providers
  repo: https://github.com/ska-sa/montblanc
- name: CUDA
  role: Architecture for the three-kernel EK -> B Sum -> Reduce workflow executed
    on NVIDIA GPUs
- name: NumPy
  role: Independent CPU reference implementation used to unit-test GPU per-antenna
    terms and chi-squared values, and to serve tiled source data
  repo: https://github.com/numpy/numpy
- name: CUB
  role: CUDA device-wide reduction primitives used for the chi-squared reduction kernel
  repo: https://github.com/nvlabs/cub
- name: HyperCube
  role: Registers problem dimensions and array schemas and provides extent_iter for
    time-dimension subdivision
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.003
    title: 'Montblanc : GPU accelerated radio interferometer measurement equations
      in support of Bayesian inference for radio observations'
  source_content_hash: sha256:5a2c2b5d29eb726257cc5614d83583b7f80c8b1c7677a88f9187669b9c7d4730
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ska-sa/montblanc
  tools:
  - CUB
  - CUDA
  - HyperCube
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
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# rime-chi-squared-visibility-computation

## Summary

Reconstruct Montblanc's three-kernel RIME pipeline (EK -> B Sum -> Reduce) that converts sky model parameters into model visibilities and a single chi-squared scalar, in CUDA on GPU or an equivalent CPU/numpy reference, for use as the likelihood in BIRO Bayesian inference.

## When to use

Reference for computing model visibilities and a single chi-squared value from a radio interferometry sky model (point/gaussian/beta sources) via the RIME, especially when unit-testing a GPU kernel against a numpy CPU reference or feeding likelihoods to Bayesian sampling.

## When NOT to use

- Data is not fully calibrated: Montblanc does not evaluate G_ps (DIE) matrices, so direction-independent gain errors must be solved elsewhere first
- The analysis requires imaging, deconvolution or source extraction rather than visibility-domain chi-squared likelihood evaluation
- Multi-GPU or distributed execution is required: only single GPUs are supported and the out-of-core BIRO solution with its data-transfer bottlenecks is not yet implemented

## Inputs

- Sky model parameters as numpy arrays: lm coordinates, Stokes parameters (IQUV), spectral indices (alpha), Gaussian shape parameters (gaussian_shape), and beta profile parameters for new morphologies
- Synthetic or observed visibility data and a weight vector
- Hypercube dimension configuration (ntime, na, nchan, npsrc/ngsrc) and array schemas
- Solver configuration, e.g. montblanc.rime_solver_cfg(dtype='double', version='tf', mem_budget=4*1024*1024*1024)

## Outputs

- Per-antenna terms A[t,p,s,lambda] from the EK kernel
- 3D model visibility array reduced from 4D per-baseline source coherencies
- Single scalar chi-squared value (per tile and/or total)
- Validation records, e.g. chi2_subdivision_check. or beta_profile_validation_report.md

## How to apply

Register a hypercube problem with dimensions such as ntime, na, nchan, npsrc, ngsrc via register_dimension and register_array calls, then serve lm coordinates, Stokes parameters, spectral indices and Gaussian shape parameters as numpy tiles sliced by context.dim_extents from provider-style data sources (e.g. point_lm, gaussian_shape). Compute the EK stage producing per-antenna terms A[t,p,s,lambda] via Equations 3 and 4, run the B Sum stage to form 4D per-baseline source coherencies reduced to a 3D model visibility array, then compute chi-squared by subtracting model from observed visibilities, squaring, multiplying by the weight vector, and summing to one scalar (mirroring slvr.solve with source_providers/sink_providers). If the problem exceeds the GPU memory budget (e.g. mem_budget=4*1024*1024*1024 in montblanc.rime_solver_cfg with dtype='double', version='tf'), subdivide along the time dimension using cube.extent_iter with iter_args like [('ntime', 100), ('nchan', 64)], issuing asynchronous transfers so data movement overlaps kernel execution, and sum the per-tile partial chi-squared values on the CPU. Validate by comparing GPU or subdivided chi-squared against an independent CPU numpy reference (optionally numexpr-accelerated) within floating-point round-off tolerance.

## Related tools

- **Montblanc** (GPU (CUDA/TensorFlow) RIME solver producing per-antenna terms, model visibilities and chi-squared; invoked via slvr.solve with source and sink providers) — https://github.com/ska-sa/montblanc
- **CUDA** (Architecture for the three-kernel EK -> B Sum -> Reduce workflow executed on NVIDIA GPUs)
- **NumPy** (Independent CPU reference implementation used to unit-test GPU per-antenna terms and chi-squared values, and to serve tiled source data) — https://github.com/numpy/numpy
- **CUB** (CUDA device-wide reduction primitives used for the chi-squared reduction kernel) — https://github.com/nvlabs/cub
- **HyperCube** (Registers problem dimensions and array schemas and provides extent_iter for time-dimension subdivision)

## Examples

```
slvr = montblanc.rime_solver_cfg(dtype='double', version='tf', mem_budget=4*1024*1024*1024); chi2 = slvr.solve(source_providers=[point_provider, gaussian_provider], sink_providers=[sink])
```

## Evaluation signals

- Subdivided chi-squared total must agree with the monolithic chi-squared within floating-point round-off tolerance (near-zero relative difference), recorded in chi2_subdivision_check.
- GPU per-antenna terms and chi2 must match an independent CPU numpy reference within floating-point tolerance across tested source/tile configurations
- Single-precision runs must match the montblanc CPU reference (rime_solver_cfg with dtype='float', version='tf') within single-precision tolerance
- Each SourceProvider's updated_dimensions() must report the correct global dimension sizes while tiles are sliced with context.dim_extents
- Chi-squared must be a single scalar obtained by subtracting model visibilities from data, squaring, applying the weight vector, and summing

## Limitations

- Only single GPUs are currently supported; multi-GPU and clustered solutions are viable via subdivision but not yet available
- An out-of-core solution for BIRO is not yet implemented, and data-transfer bottlenecks for distributed BIRO remain unsolved
- E_ps (DDE) terms are substituted with analytic expressions rather than full matrix evaluation, and G_ps (DIE) matrix evaluation is not included since BIRO operates on fully calibrated data
- Other source morphologies such as beta profiles must be added by extending the EK kernel following the GaussianSourceProvider pattern

## Evidence

- [intro] The model visibilities are subtracted from the data, squared, and multiplied by the weight vector, then summed to produced a single scalar χ2 value.: [evidence span withheld — anchor status: not_found]
- [discussion] Expensive, per-antenna terms are computed in an EK kernel, and output to the GPU's main memory.: [evidence span withheld — anchor status: not_found]
