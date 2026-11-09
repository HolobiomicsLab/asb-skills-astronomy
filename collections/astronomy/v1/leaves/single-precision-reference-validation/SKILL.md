---
name: single-precision-reference-validation
description: Use when a GPU/CUDA scientific kernel (e.g. Montblanc's EK, B Sum and
  reduction kernels) must be verified against an independent CPU reference implementation,
  especially when outputs are single-precision floats like χ2.
when_to_use_negative:
- 'The data require direction-dependent effects: Montblanc omits evaluation of G_ps
  (DIE) matrices and substitutes analytic expressions for the E_ps (DDE) terms, operating
  on fully calibrated data only'
- The target accuracy exceeds single-precision floating-point tolerance; the reference
  match is only meaningful at dtype='float' tolerance
- 'The problem must be distributed or multi-GPU: only single GPUs are currently supported
  in this implementation'
tools:
- name: Montblanc
  role: GPU RIME implementation whose three CUDA kernels (EK, B Sum, reduction) are
    validated against the NumPy CPU reference
  repo: https://github.com/ska-sa/montblanc
- name: NumPy
  role: CPU reference implementation of the EK, B Sum and chi2 reduction stages for
    single-precision comparison
  repo: https://github.com/numpy/numpy
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.003
    title: 'Montblanc : GPU accelerated radio interferometer measurement equations
      in support of Bayesian inference for radio observations'
  source_content_hash: sha256:7a8cc67c7d42c57933d922fba9dbf0190b7b0fdd78f062d7df11aca9ee3acc07
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ska-sa/montblanc
  tools:
  - Montblanc
  - NumPy
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# single-precision-reference-validation

## Summary

Validates a GPU-accelerated RIME pipeline (Montblanc) by re-implementing its three-kernel workflow — EK per-antenna terms, B Sum source coherencies, and χ2 reduction — in CPU NumPy at single precision, and confirming the scalar χ2 matches the GPU solver within floating-point tolerance.

## When to use

Use when a GPU/CUDA scientific kernel (e.g. Montblanc's EK, B Sum and reduction kernels) must be verified against an independent CPU reference implementation, especially when outputs are single-precision floats like χ2.

## When NOT to use

- The data require direction-dependent effects: Montblanc omits evaluation of G_ps (DIE) matrices and substitutes analytic expressions for the E_ps (DDE) terms, operating on fully calibrated data only
- The target accuracy exceeds single-precision floating-point tolerance; the reference match is only meaningful at dtype='float' tolerance
- The problem must be distributed or multi-GPU: only single GPUs are currently supported in this implementation

## Inputs

- Synthetic single-precision (float32) sky-model arrays: lm coordinates, Stokes parameters, spectral indices, Gaussian shape parameters
- Synthetic observed visibilities and weight vector
- Montblanc hypercube with registered dimensions (ntime, na, nchan, npsrc=10, ngsrc=10) and array schemas
- Montblanc CPU reference solver configuration (rime_solver_cfg, dtype='float', version='tf')

## Outputs

- Per-antenna terms A[t, p, s, lambda] from the NumPy EK stage
- 3D model visibility array from the NumPy B Sum stage
- Single-scalar chi2 value from the NumPy pipeline
- Matching chi2 scalar from Montblanc (float dtype, tf version) for tolerance comparison

## How to apply

Register a small test hypercube (e.g. 10 point sources 'npsrc' and 10 Gaussian sources 'ngsrc' with nominal time, antenna and channel dimensions) via the hypercube's register_dimension and register_array calls. Generate synthetic lm coordinates, Stokes parameters, spectral indices and Gaussian shape parameters in single precision (float32), serving them through PointSourceProvider- and GaussianSourceProvider-style data sources that slice tiles via context.dim_extents. Reproduce the GPU pipeline in NumPy: compute per-antenna terms A[t,p,s,lambda] (EK stage, Equations 3–4), combine them into per-baseline 4D source coherencies and a 3D model visibility array (B Sum stage), then subtract model from observed visibilities, square, apply the weight vector and sum to a single χ2 scalar. Run Montblanc's CPU reference via rime_solver_cfg with dtype='float' and version='tf', invoking slvr.solve(source_providers=..., sink_providers=...), and compare the two χ2 values within single-precision tolerance.

## Related tools

- **Montblanc** (GPU RIME implementation whose three CUDA kernels (EK, B Sum, reduction) are validated against the NumPy CPU reference) — https://github.com/ska-sa/montblanc
- **NumPy** (CPU reference implementation of the EK, B Sum and chi2 reduction stages for single-precision comparison) — https://github.com/numpy/numpy

## Examples

```
cfg = montblanc.rime_solver_cfg(dtype='float', version='tf'); slvr = montblanc.RimeSolver.factory(cfg); chi2 = slvr.solve(source_providers=[...], sink_providers=[...])
```

## Evaluation signals

- NumPy chi2 scalar matches the Montblanc CPU reference chi2 (dtype='float', version='tf') within single-precision floating-point tolerance
- NumPy model visibilities have shape [ntime, nbl, nchan] (3D) after reduction from the 4D [time, baseline, source, channel] source-coherency array
- Per-antenna terms A[t, p, s, lambda] cover all time, antenna, source and channel extents; source counts match hypercube registration (10 point, 10 Gaussian sources)
- Solver call completes with the same source_providers/sink_providers interface used by the GPU path, producing a single scalar not a vector
- Slicing via context.dim_extents yields tiles that reconstruct the full hypercube without gaps or overlaps

## Limitations

- Only single GPUs are supported; multi-GPU and clustered solutions are not yet available
- Analytic expressions substitute for full E_ps (DDE) matrix evaluation, so the reference only validates that reduced model
- The dominant kernel is theoretically memory bound but in practice balanced between compute and memory due to L1/L2 cache retention, so performance-based validation is unreliable

## Evidence

- [intro] chi2 reduction correctness: [evidence span withheld — anchor status: not_found]
- [discussion] EK and B Sum kernel decomposition: [evidence span withheld — anchor status: not_found]
- [discussion] DDE/DIE scope limitation: [evidence span withheld — anchor status: not_found]
