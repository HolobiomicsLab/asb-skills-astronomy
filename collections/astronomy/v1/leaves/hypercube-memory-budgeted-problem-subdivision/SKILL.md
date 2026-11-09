---
name: hypercube-memory-budgeted-problem-subdivision
description: Use when a RIME (or similarly dimension-parallel) GPU computation exceeds
  a supplied memory budget and must still yield one χ2 value; triggered by large ntime/nchan/npsrc
  hypercube dimensions that force time-dimension tiling with asynchronous transfers.
when_to_use_negative:
- The RIME problem already fits within the GPU memory budget, so a single monolithic
  slvr.solve evaluation suffices and subdivision only adds overhead
- The target workload requires direction-dependent effect (DDE) matrix evaluation
  or G_ps (DIE) terms, which Montblanc does not include (analytic substitutions only)
- Multi-GPU or clustered execution is required — only single GPUs are currently supported
  and the out-of-core BIRO solution is not yet implemented
tools:
- name: Montblanc
  role: TensorFlow-based RIME solver whose memory-budget subdivision and HyperCube
    extent iteration this skill reproduces
  repo: https://github.com/ska-sa/montblanc
- name: Python
  role: Host language for the solver configuration, SourceProvider data sources, and
    χ2 accumulation script
- name: NumPy
  role: Stores the underlying visibility and sky-model arrays that SourceProvider
    methods slice with context.dim_extents
- name: CUDA
  role: GPU execution architecture underlying the RIME kernels into which tiles are
    asynchronously transferred
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.003
    title: 'Montblanc : GPU accelerated radio interferometer measurement equations
      in support of Bayesian inference for radio observations'
  source_content_hash: sha256:942dfaaacb9ccc54f7a80ed2a0a08ae0870b69eac24486893e0d9ab4019dc577
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# hypercube-memory-budgeted-problem-subdivision

## Summary

Reconstructs Montblanc's mechanism for evaluating a RIME problem larger than the GPU memory budget by subdividing it along the time dimension into tiles, feeding each tile to a solver via HyperCube extent iteration and asynchronous data transfers, and summing per-tile partial χ2 values into a single scalar.

## When to use

Use when a RIME (or similarly dimension-parallel) GPU computation exceeds a supplied memory budget and must still yield one χ2 value; triggered by large ntime/nchan/npsrc hypercube dimensions that force time-dimension tiling with asynchronous transfers.

## When NOT to use

- The RIME problem already fits within the GPU memory budget, so a single monolithic slvr.solve evaluation suffices and subdivision only adds overhead
- The target workload requires direction-dependent effect (DDE) matrix evaluation or G_ps (DIE) terms, which Montblanc does not include (analytic substitutions only)
- Multi-GPU or clustered execution is required — only single GPUs are currently supported and the out-of-core BIRO solution is not yet implemented

## Inputs

- RIME problem hypercube dimensions (ntime, na, nchan, npsrc) registered on a HyperCube
- Underlying numpy arrays of visibility data, uvw coordinates, sky-model source data (lm, stokes, alpha, gaussian shape terms) and frequencies
- Memory budget in bytes (e.g. mem_budget=4*1024*1024*1024) supplied to the solver configuration
- Tile extents produced by cube.extent_iter with iter_args such as [('ntime', 100), ('nchan', 64)]

## Outputs

- A single total χ2 scalar summed from per-tile partial χ2 values
- Per-tile (lower, upper) time extents written to chi2_subdivision_check. alongside the monolithic χ2 value
- Relative-difference metric comparing subdivided vs monolithic χ2 totals

## How to apply

Configure the solver with montblanc.rime_solver_cfg(dtype='double', version='tf', mem_budget=4*1024*1024*1024) and first compute a monolithic χ2 over the full problem with slvr.solve using configured source and sink providers. Register hypercube dimensions (ntime, na, nchan, npsrc) and array schemas on a HyperCube (e.g. cube.register_dimension('ntime', 10000, description='Timesteps')), then iterate over time tiles with cube.extent_iter using iter_args such as [('ntime', 100), ('nchan', 64)] to obtain (lower, upper) extents per tile. For each tile, supply data through a SourceProvider whose methods (e.g. lm, point_stokes, frequency) slice underlying numpy arrays via context.dim_extents and whose updated_dimensions reports global dimension sizes, issuing asynchronous transfers so data movement overlaps kernel execution. Per tile, compute a partial χ2 by differencing model visibilities from observed visibilities, squaring, multiplying by the weight vector, and reducing to a scalar; sum these partials into the subdivided total.

## Related tools

- **Montblanc** (TensorFlow-based RIME solver whose memory-budget subdivision and HyperCube extent iteration this skill reproduces) — https://github.com/ska-sa/montblanc
- **Python** (Host language for the solver configuration, SourceProvider data sources, and χ2 accumulation script)
- **NumPy** (Stores the underlying visibility and sky-model arrays that SourceProvider methods slice with context.dim_extents)
- **CUDA** (GPU execution architecture underlying the RIME kernels into which tiles are asynchronously transferred)

## Examples

```
cfg = montblanc.rime_solver_cfg(dtype='double', version='tf', mem_budget=4*1024*1024*1024)
```

## Evaluation signals

- The subdivided χ2 total must agree with the monolithic χ2 within floating-point round-off tolerance (near-zero relative difference)
- Per-tile SourceProvider methods must report the global dimension sizes in updated_dimensions, not the tile-local sizes, so the solver sees consistent global shape
- Every (lower, upper) extent from cube.extent_iter must cover ntime exactly once with no gaps or overlaps across tiles
- Each partial χ2 is computed by subtracting model from observed visibilities, squaring, multiplying by the weight vector, and reducing to a single scalar before summation
- Both the monolithic and subdivided χ2 values are written to chi2_subdivision_check. for comparison

## Limitations

- Only single GPUs are currently supported; multi-GPU and clustered solutions are not yet available
- An out-of-core solution for BIRO is not yet implemented, and data-transfer bottlenecks inherent to the subdivided/distributed solution remain unsolved
- Only time-dimension subdivision is described; other dimensions (channels, sources) are not subdivided in this mechanism

## Evidence

- [intro] chi2 reduction from visibilities: [evidence span withheld — anchor status: not_found]
