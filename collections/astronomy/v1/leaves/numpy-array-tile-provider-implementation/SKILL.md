---
name: numpy-array-tile-provider-implementation
description: Use when a Montblanc RIME problem exceeds the configured mem_budget and
  must be iterated over time tiles via HyperCube.extent_iter, requiring per-tile data
  sources backed by NumPy arrays.
when_to_use_negative:
- The full RIME problem already fits within the GPU memory budget, in which case a
  monolithic slvr.solve over the whole problem is simpler and sufficient
- Data lives in a measurement-set or disk-backed store requiring a non-NumPy source
  backend rather than in-memory array slicing
- The task needs direction-dependent effect (DDE) matrix evaluation or multi-GPU distribution,
  which Montblanc does not implement (analytic expressions substituted, single GPU
  only)
edam_operation: http://edamontology.org/operation_2409
tools:
- name: Montblanc
  role: GPU RIME solver framework whose SourceProvider/HyperCube APIs the tile provider
    implements; solver configured via montblanc.rime_solver_cfg(dtype='double', version='tf',
    mem_budget=4*1024*1024*1024)
  repo: https://github.com/ska-sa/montblanc
- name: Python
  role: Implementation language for the SourceProvider data-source methods and tile
    iteration loop
  repo: https://github.com/ska-sa/montblanc
- name: NumPy
  role: Holds the full untiled arrays; provider methods slice them per context.dim_extents
    for each tile
  repo: https://github.com/numpy/numpy
- name: CUDA
  role: Underlying GPU architecture receiving the asynchronous tile transfers that
    overlap kernel execution
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.003
    title: 'Montblanc : GPU accelerated radio interferometer measurement equations
      in support of Bayesian inference for radio observations'
  source_content_hash: sha256:ad8f4e6fdbb50e163e282e435871e3538b65d998e67f5993f729fea1048036dc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ska-sa/montblanc
  tools:
  - CUDA
  - Montblanc
  - NumPy
  - Python
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# NumPy array tile provider implementation

## Summary

Implements Montblanc-style SourceProvider data sources that serve (lower, upper)-extented tiles of in-memory NumPy arrays to GPU RIME solvers, so a problem too large for the GPU memory budget can be subdivided along the time dimension while asynchronous transfers overlap kernel execution.

## When to use

Use when a Montblanc RIME problem exceeds the configured mem_budget and must be iterated over time tiles via HyperCube.extent_iter, requiring per-tile data sources backed by NumPy arrays.

## When NOT to use

- The full RIME problem already fits within the GPU memory budget, in which case a monolithic slvr.solve over the whole problem is simpler and sufficient
- Data lives in a measurement-set or disk-backed store requiring a non-NumPy source backend rather than in-memory array slicing
- The task needs direction-dependent effect (DDE) matrix evaluation or multi-GPU distribution, which Montblanc does not implement (analytic expressions substituted, single GPU only)

## Inputs

- In-memory NumPy arrays for RIME inputs (lm coordinates, frequency, point/gaussian lm, stokes, alpha, gaussian_shape, uvw, observed visibilities)
- HyperCube with registered dimensions (ntime, na, nchan, npsrc) and array schemas
- Tile iteration arguments, e.g. [('ntime', 100), ('nchan', 64)]

## Outputs

- Per-tile slices of the underlying NumPy arrays issued via asynchronous transfers into the solver
- A partial chi-squared scalar per time tile, summed into a single subdivided total chi-squared value

## How to apply

Register hypercube dimensions (e.g. ntime=10000, na, nchan, npsrc) and array schemas on a HyperCube, then obtain per-tile extents by iterating with cube.extent_iter and iter_args such as [('ntime', 100), ('nchan', 64)]. Implement a SourceProvider whose data-source methods (lm, frequency, point_lm, point_stokes, point_alpha, gaussian_lm, gaussian_stokes, gaussian_alpha, gaussian_shape) slice the underlying NumPy arrays using context.dim_extents so only the current tile is issued as an asynchronous transfer to the solver, while updated_dimensions reports the global (untiled) dimension sizes so the solver indexes correctly. For each time tile, the solver computes a partial chi-squared by differencing model from observed visibilities, squaring, weighting, and reducing to a scalar; partial values are summed on the CPU into a single subdivided total. Rationale: Montblanc subdivides the problem along the time dimension when it exceeds the supplied memory budget, and tile-level providers let data movement overlap kernel execution.

## Related tools

- **Montblanc** (GPU RIME solver framework whose SourceProvider/HyperCube APIs the tile provider implements; solver configured via montblanc.rime_solver_cfg(dtype='double', version='tf', mem_budget=4*1024*1024*1024)) — https://github.com/ska-sa/montblanc
- **Python** (Implementation language for the SourceProvider data-source methods and tile iteration loop) — https://github.com/ska-sa/montblanc
- **NumPy** (Holds the full untiled arrays; provider methods slice them per context.dim_extents for each tile) — https://github.com/numpy/numpy
- **CUDA** (Underlying GPU architecture receiving the asynchronous tile transfers that overlap kernel execution)

## Examples

```
slvr = montblanc.rime_solver_cfg(dtype='double', version='tf', mem_budget=4*1024*1024*1024); for (lt, ut) in cube.extent_iter([('ntime', 100), ('nchan', 64)]): slvr.solve(source=provider, sink=sink)
```

## Evaluation signals

- Subdivided total chi-squared equals the monolithic chi-squared from a full-problem slvr.solve within floating-point round-off (near-zero relative difference), recorded in chi2_subdivision_check.
- Each provider method returns array slices whose shape matches the tile's context.dim_extents, and updated_dimensions reports the global dimension sizes (e.g. ntime=10000), not tile-local sizes
- extent_iter over [('ntime', 100), ('nchan', 64)] yields (lower, upper) extents that cover the registered ntime dimension exactly once with no gaps or overlaps
- Per-tile chi-squared is computed by the documented reduction: model visibilities subtracted from observed, squared, multiplied by the weight vector, then summed to a scalar

## Limitations

- Only single GPUs are supported; multi-GPU and clustered solutions are not yet available, though subdivision makes them viable
- An out-of-core solution for BIRO is not implemented, and data-transfer bottlenecks for subdivided/distributed execution remain unsolved
- G_ps (DIE) matrix evaluation is omitted (BIRO operates on fully calibrated data) and E_ps (DDE) terms use analytic expressions rather than full matrix evaluation

## Evidence

- [intro] chi-squared reduction: [evidence span withheld — anchor status: not_found]
