---
name: hypercube-dimension-array-registration
description: Use when constructing a Montblanc solver problem (e.g. adding new source
  types or testing the EK/B Sum pipeline) and you must define named dimensions, extents,
  and the arrays tiled by SourceProvider data sources.
when_to_use_negative:
- Input visibilities come from a MeasurementSet calibration context needing MeqTrees
  or CASA ML rather than a Montblanc GPU solver problem definition
- You only need to evaluate the RIME analytically in numpy without a GPU solver —
  dimension registration adds no value
- The problem requires DDE/DIE (G_ps/E_ps) matrix evaluation, which Montblanc does
  not include (analytic E_ps substitutions only)
tools:
- name: Montblanc
  role: Provides the hypercube register_dimension/register_array API consumed by SourceProvider
    data sources before slvr.solve
  repo: https://github.com/ska-sa/montblanc
- name: Python
  role: Language in which dimension registration and provider data sources are written
- name: NumPy
  role: Generates and tiles the synthetic arrays (lm, stokes, alpha, shape) served
    through dim_extents slices
  repo: numpy/numpy
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.003
    title: 'Montblanc : GPU accelerated radio interferometer measurement equations
      in support of Bayesian inference for radio observations'
  source_content_hash: sha256:e5c2f98bff8d96e05c705c6ea8b44cbbf35b3adc25b6dcce681a4f08f121d55f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ska-sa/montblanc
  tools:
  - Montblanc
  - NumPy
  - Python
  - TensorFlow
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# hypercube-dimension-array-registration

## Summary

Register the named problem dimensions (e.g. ntime, npsrc, ngsrc) and array schemas on a Montblanc hypercube so that data-source providers can slice input tiles (lm, stokes, alpha, gaussian shape) to the solver's dim_extents before GPU RIME evaluation.

## When to use

Use when constructing a Montblanc solver problem (e.g. adding new source types or testing the EK/B Sum pipeline) and you must define named dimensions, extents, and the arrays tiled by SourceProvider data sources.

## When NOT to use

- Input visibilities come from a MeasurementSet calibration context needing MeqTrees or CASA ML rather than a Montblanc GPU solver problem definition
- You only need to evaluate the RIME analytically in numpy without a GPU solver — dimension registration adds no value
- The problem requires DDE/DIE (G_ps/E_ps) matrix evaluation, which Montblanc does not include (analytic E_ps substitutions only)

## Inputs

- Named problem dimensions (ntime, na, nchan, npsrc, ngsrc) with descriptions
- Array schemas for lm coordinates, Stokes parameters, spectral indices and shape parameters
- Synthetic single- or double-precision numpy input data served by SourceProvider-style data sources

## Outputs

- A registered hypercube whose dimensions and arrays back montblanc.rime_solver_cfg solver instances
- dim_extents context objects used by data-source/sink providers to tile numpy inputs

## How to apply

Build a hypercube and call register_dimension for each problem axis — e.g. cube.register_dimension("ntime", 10000, description="Timesteps") — along with source-count dimensions such as 'npsrc' and 'ngsrc', then register_array for each schema (lm coordinates, Stokes parameters, spectral indices, Gaussian/beta shape parameters). Register the arrays in single or double precision matching the solver configuration chosen via montblanc.rime_solver_cfg(dtype='double'|'float', version='tf', mem_budget=4*1024*1024*1024). Data sources (lm, frequency, point_lm, point_stokes, point_alpha, gaussian_lm, gaussian_stokes, gaussian_alpha, gaussian_shape) then slice tiles via context.dim_extents, so extents must be declared before providers run. When adding a new morphology such as a beta profile, register its count dimension ('nsrc'/'ngsrc'-style extent) and report updated_dimensions() from the provider so tile slicing stays consistent. If the declared problem exceeds the GPU memory budget, the solver subdivides along the time dimension, so honest extent declarations determine chunking behaviour.

## Related tools

- **Montblanc** (Provides the hypercube register_dimension/register_array API consumed by SourceProvider data sources before slvr.solve) — https://github.com/ska-sa/montblanc
- **Python** (Language in which dimension registration and provider data sources are written)
- **NumPy** (Generates and tiles the synthetic arrays (lm, stokes, alpha, shape) served through dim_extents slices) — numpy/numpy

## Examples

```
cube.register_dimension("ntime", 10000, description="Timesteps")
```

## Evaluation signals

- Every data source (point_lm, gaussian_shape, etc.) can slice a tile using context.dim_extents without KeyError/shape mismatch
- Solver accepts the registered cube in slvr.solve(source_providers=..., sink_providers=...) with the configured dtype ('float' or 'double')
- Provider updated_dimensions() correctly reports new source counts (e.g. beta profile counts) after registration
- Problems larger than mem_budget (e.g. 4*1024*1024*1024) are subdivided along ntime without extent errors

## Limitations

- Only single-GPU problems are currently supported; dimension subdivision enables but does not yet implement multi-GPU or clustered solutions
- An out-of-core BIRO solution is not implemented, so registered extents must fit the supplied GPU memory budget
- Registration only covers morphologies implemented in the kernel; beta profiles were noted as future additions in the paper

## Evidence

- [discussion] It is architected to allow addition of other source morphologies, such as beta profiles: [evidence span withheld — anchor status: not_found]
