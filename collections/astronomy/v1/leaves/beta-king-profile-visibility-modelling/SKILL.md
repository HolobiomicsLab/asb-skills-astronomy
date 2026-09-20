---
name: beta-king-profile-visibility-modelling
description: Use when a sky model requires beta-profile (King) sources in Montblanc's
  RIME evaluation and GPU-computed per-antenna terms and chi2 must be verified against
  a trusted CPU reference.
when_to_use_negative:
- When only point or gaussian source morphologies are needed — Montblanc's existing
  point_lm/gaussian_* data sources already cover those without kernel extension
- When the RIME must evaluate direction-dependent gain matrices (G_ps DIEs) — the
  paper notes these are not included since BIRO operates on fully calibrated data
- When multi-GPU or distributed evaluation is required — only single GPUs are currently
  supported
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Montblanc
  role: GPU RIME solver whose EK kernel is extended with the beta-profile source loop;
    provides the solver configuration and data source/sink provider framework
  repo: https://github.com/ska-sa/montblanc
- name: NumPy
  role: Supplies numpy tiles for the BetaSourceProvider and implements the independent
    CPU reference implementation for validation
  repo: https://github.com/numpy/numpy
- name: CUDA
  role: NVIDIA architecture on which the extended EK kernel and B Sum kernel run
- name: TensorFlow
  role: Backend version ('tf') of the Montblanc solver used to evaluate the RIME with
    the new beta source provider
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.003
    title: 'Montblanc : GPU accelerated radio interferometer measurement equations
      in support of Bayesian inference for radio observations'
  source_content_hash: sha256:b07510beb83fe24b8501cec49698470e5f85d2b6a9a9e1455910cb5c988310c3
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
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# beta-king-profile-visibility-modelling

## Summary

Extends Montblanc's GPU RIME pipeline with a beta-profile (King 1966/1972) source morphology by adding a dedicated source loop to the EK kernel, following the existing point and gaussian source loops, and validating the resulting per-antenna terms and chi-squared against an independent CPU/numpy reference.

## When to use

Use when a sky model requires beta-profile (King) sources in Montblanc's RIME evaluation and GPU-computed per-antenna terms and chi2 must be verified against a trusted CPU reference.

## When NOT to use

- When only point or gaussian source morphologies are needed — Montblanc's existing point_lm/gaussian_* data sources already cover those without kernel extension
- When the RIME must evaluate direction-dependent gain matrices (G_ps DIEs) — the paper notes these are not included since BIRO operates on fully calibrated data
- When multi-GPU or distributed evaluation is required — only single GPUs are currently supported

## Inputs

- Beta-profile source parameters: lm coordinates, stokes fluxes, spectral indices, and beta shape parameters as numpy arrays tiled and sliced to the solver's dim_extents
- Observed visibility data with weights for chi-squared computation
- Solver configuration: dtype='double', version='tf', mem_budget (e.g. 4*1024*1024*1024 bytes)

## Outputs

- Per-antenna terms for beta-profile sources computed by the extended EK kernel
- Model visibilities formed by the B Sum kernel from per-antenna terms
- Scalar chi-squared value (weighted sum of squared visibility residuals) used by BIRO
- beta_profile_validation_report.md recording max deviations between GPU and CPU reference

## How to apply

Clone the Montblanc repository and install it in a virtual environment per the documented pip procedure (e.g. pip install git+git://github.com/ska-sa/montblanc.git@master). Implement a beta-profile source loop inside the EK kernel so per-antenna terms are computed for beta sources in the same framework as the existing point and gaussian source loops, mirroring the paper's design where expensive per-antenna terms are computed in an EK kernel and combined by the B Sum kernel. Write a BetaSourceProvider following the GaussianSourceProvider pattern, supplying lm coordinates, stokes parameters, spectral indices and beta shape parameters as numpy tiles sliced to the context's dim_extents (nsrc/ngsrc-style extents) and reporting the beta source count through updated_dimensions(). Configure the solver with montblanc.rime_solver_cfg(dtype='double', version='tf', mem_budget=4*1024*1024*1024) and run slvr.solve with the beta provider registered alongside the source/sink provider lists. Implement an independent CPU/numpy (optionally numexpr-accelerated) reference computing beta-profile per-antenna terms and chi2 — subtracting model visibilities from observed data, squaring, multiplying by weights, summing to a scalar — and compare GPU outputs across test extents, recording max deviations.

## Related tools

- **Montblanc** (GPU RIME solver whose EK kernel is extended with the beta-profile source loop; provides the solver configuration and data source/sink provider framework) — https://github.com/ska-sa/montblanc
- **NumPy** (Supplies numpy tiles for the BetaSourceProvider and implements the independent CPU reference implementation for validation) — https://github.com/numpy/numpy
- **CUDA** (NVIDIA architecture on which the extended EK kernel and B Sum kernel run)
- **TensorFlow** (Backend version ('tf') of the Montblanc solver used to evaluate the RIME with the new beta source provider)

## Examples

```
cfg = montblanc.rime_solver_cfg(dtype='double', version='tf', mem_budget=4*1024*1024*1024); slvr.solve(src_providers=[BetaSourceProvider(...)], sink_providers=[...])
```

## Evaluation signals

- GPU per-antenna terms for beta-profile sources match the CPU/numpy reference within floating-point tolerance across tested source/tile configurations
- GPU chi-squared equals the CPU reference chi2 (data minus model, squared, weight-multiplied, summed to a scalar) within floating-point tolerance
- BetaSourceProvider correctly slices numpy tiles to the context's dim_extents (nsrc/ngsrc-style) and reports beta source count via updated_dimensions()
- Solver runs end-to-end with montblanc.rime_solver_cfg(dtype='double', version='tf') and the beta provider registered without errors
- Maximum deviations between GPU and CPU are recorded in beta_profile_validation_report.md

## Limitations

- Beta profiles are mentioned in the paper as a future addition ('It is architected to allow addition of other source morphologies, such as beta profiles') — the loop must be added by the practitioner
- Only single GPUs are supported; the problem is subdivided along the time dimension when it exceeds the memory budget
- E_ps (DDE) terms are substituted with analytic expressions rather than full matrix evaluation, and G_ps (DIE) matrices are not included

## Evidence

- [discussion] It is architected to allow addition of other source morphologies, such as beta profiles: [evidence span withheld — anchor status: not_found]
- [discussion] EK kernel computes per-antenna terms: [evidence span withheld — anchor status: not_found]
- [intro] chi-squared reduction definition: [evidence span withheld — anchor status: not_found]
- [methods] filter step: gaussian data source pattern: "def gaussian_lm(self, context):
            """ Gaussian lm coordinate data source """"
- [other] solver configuration from workflow: "montblanc.rime_solver_cfg(dtype='double', version='tf', mem_budget=4*1024*1024*1024)"
