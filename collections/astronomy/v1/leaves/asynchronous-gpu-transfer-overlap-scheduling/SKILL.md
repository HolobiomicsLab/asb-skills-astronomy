---
name: asynchronous-gpu-transfer-overlap-scheduling
description: Use when the RIME problem (ntime × na × nchan × npsrc arrays) is too
  large to fit in a GPU memory budget (e.g. mem_budget=4GB) yet a single χ² scalar
  must still be produced.
when_to_use_negative:
- The problem already fits within the GPU memory budget — compute the monolithic χ²
  directly instead of incurring tiling/transfer overhead
- Data must be shared across the time dimension (e.g. direction-dependent G_ps DIE
  matrix evaluation), which Montblanc does not support — only single GPUs are supported
  and G_ps (DIE) matrices are not evaluated
- The target is imaging/deconvolution or sky-model extraction rather than iterative
  visibility-domain χ² evaluation for Bayesian inference (BIRO)
edam_operation: http://edamontology.org/operation_0292
tools:
- name: Montblanc
  role: GPU RIME solver whose memory-budget subdivision, HyperCube extent iteration
    and SourceProvider slicing implement the async transfer/overlap scheduling
  repo: https://github.com/ska-sa/montblanc
- name: HyperCube
  role: Registers dimensions (ntime, na, nchan, npsrc) and array schemas; extent_iter
    yields (lower, upper) time tiles for subdivision
  repo: https://github.com/ska-sa/montblanc
- name: CUDA
  role: GPU execution architecture on which the EK and B Sum kernels run while transfers
    are overlapped
- name: NumPy
  role: Holds the underlying full-problem arrays that the SourceProvider slices per
    tile via context.dim_extents
  repo: numpy/numpy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.003
    title: 'Montblanc : GPU accelerated radio interferometer measurement equations
      in support of Bayesian inference for radio observations'
  source_content_hash: sha256:f62b8030b5d9d6eadf9a400faa0eabfc60f079dde5cb273e18fc5b816bb01b4b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ska-sa/montblanc
  tools:
  - CUDA
  - HyperCube
  - Montblanc
  - NumPy
  - Python
  edam_operation: http://edamontology.org/operation_0292
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# asynchronous-gpu-transfer-overlap-scheduling

## Summary

Schedules RIME solver work so that host-to-device data transfers overlap GPU kernel execution: when the problem exceeds the GPU memory budget, it is subdivided along the time dimension into tiles processed by solvers with asynchronous transfers, producing per-tile partial χ² values that sum to a single total χ².

## When to use

Use when the RIME problem (ntime × na × nchan × npsrc arrays) is too large to fit in a GPU memory budget (e.g. mem_budget=4GB) yet a single χ² scalar must still be produced.

## When NOT to use

- The problem already fits within the GPU memory budget — compute the monolithic χ² directly instead of incurring tiling/transfer overhead
- Data must be shared across the time dimension (e.g. direction-dependent G_ps DIE matrix evaluation), which Montblanc does not support — only single GPUs are supported and G_ps (DIE) matrices are not evaluated
- The target is imaging/deconvolution or sky-model extraction rather than iterative visibility-domain χ² evaluation for Bayesian inference (BIRO)

## Inputs

- RIME problem arrays (ntime, na, nchan, npsrc dimensions) held in numpy arrays
- Solver configuration (dtype, version='tf', mem_budget)
- Source and sink providers (e.g. lm, frequency, point_lm, point_stokes, point_alpha data sources; complex_phase sink)
- Observed visibilities and weight vector for χ² computation

## Outputs

- Per-tile partial χ² scalars
- A single subdivided total χ² value
- chi2_subdivision_check.json recording subdivided and monolithic χ² for comparison

## How to apply

Configure the solver via montblanc.rime_solver_cfg(dtype='double', version='tf', mem_budget=4*1024*1024*1024) and compute the monolithic χ² first as a reference. Register hypercube dimensions (ntime, na, nchan, npsrc) and array schemas on a HyperCube, then iterate over time tiles with cube.extent_iter using iter_args such as [("ntime", 100), ("nchan", 64)] to obtain (lower, upper) extents per tile. For each tile, a SourceProvider slices the underlying numpy arrays using context.dim_extents and reports the global dimension sizes in updated_dimensions, issuing asynchronous transfers so data movement overlaps kernel execution rather than serialising against it. Per tile, compute the partial χ² by differencing model from observed visibilities, squaring, weighting, and reducing to a scalar, then accumulate the per-tile values on the CPU into one subdivided total. The rationale is that time is the subdivisible dimension: sources/baselines within a timestep are independent, so tiling time preserves the χ² sum exactly while keeping each tile within the memory budget.

## Related tools

- **Montblanc** (GPU RIME solver whose memory-budget subdivision, HyperCube extent iteration and SourceProvider slicing implement the async transfer/overlap scheduling) — https://github.com/ska-sa/montblanc
- **HyperCube** (Registers dimensions (ntime, na, nchan, npsrc) and array schemas; extent_iter yields (lower, upper) time tiles for subdivision) — https://github.com/ska-sa/montblanc
- **CUDA** (GPU execution architecture on which the EK and B Sum kernels run while transfers are overlapped)
- **NumPy** (Holds the underlying full-problem arrays that the SourceProvider slices per tile via context.dim_extents) — numpy/numpy

## Examples

```
cfg = montblanc.rime_solver_cfg(dtype='double', version='tf', mem_budget=4*1024*1024*1024); tiles = cube.extent_iter([("ntime", 100), ("nchan", 64)])
```

## Evaluation signals

- The subdivided χ² total must agree with the monolithic χ² computed over the full problem within floating-point round-off (near-zero relative difference)
- Every extent pair (lower, upper) from extent_iter must tile the global ntime range without gaps or overlaps
- updated_dimensions on the SourceProvider must report the global (not per-tile) dimension sizes so solvers see a consistent hypercube
- Per-tile data slices obtained via context.dim_extents must match the (lower, upper) extents requested in iter_args
- Peak GPU memory use per solver must remain within the configured mem_budget (e.g. 4*1024*1024*1024 bytes)

## Limitations

- Only single GPUs are currently supported; multi-GPU and clustered solutions are not yet available, although subdivision makes them viable
- An out-of-core solution for BIRO is not yet implemented and is required for multi-GPU/distributed computation
- Data-transfer bottlenecks inherent to the subdivided/distributed solution remain unsolved
- Evaluation of G_ps (DIE) matrices is not included (BIRO operates on fully calibrated data), and E_ps (DDE) terms are substituted with analytic expressions

## Evidence

- [intro] time subdivision trigger: "if the problem is too large to fit into a supplied memory budget, the problem is subdivided along the time dimension."
- [abstract] GPU transfer between iterations: "Modified model parameters are transferred to the GPU between each iteration."
- [methods] hypercube dimension registration: "cube.register_dimension("ntime", 10000, description="Timesteps")"
- [intro] chi2 reduction definition: [evidence span withheld — anchor status: not_found]
- [discussion] single-GPU limitation: "While only single GPUs are currently supported, this subdivision makes multi-GPU and clustered solutions to the RIME viable."
