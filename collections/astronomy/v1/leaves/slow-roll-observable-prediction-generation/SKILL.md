---
name: slow-roll-observable-prediction-generation
description: Use when theoretical predictions of slow-roll single-field inflation
  models (spectral observables over a model prior/reheating grid) are needed as a
  forward model for CMB comparison or Bayesian model selection.
when_to_use_negative:
- The model of interest is not among the 74 inflationary models implemented in ASPIC
  and you do not intend to extend the code
- The task is non-slow-roll or multi-field inflation, outside the single-field slow-roll
  class ASPIC covers
- You only need parameter inference for a single already-fixed model rather than predictions
  across a model prior grid
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: ASPIC
  role: Publicly available runtime library providing all routines needed to quickly
    derive reheating consistent observable predictions for slow-roll single field
    inflation models
  repo: https://github.com/cosmicinflation/aspic
- name: libaspic
  role: Shared scientific library of fast modern Fortran routines installed via './configure',
    'make', 'make install'; documented via 'man libaspic'
  repo: https://github.com/cosmicinflation/aspic
- name: gfortran
  role: Open source Fortran compiler supporting the FORTRAN08 complex atan() intrinsic
    calls used by ASPIC
- name: OpenMP
  role: Parallel processing of model predictions, switched by default using the '-fopenmp'
    flag; can be disabled with --disable-fopenmp
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.dark.2014.01.003
    title: Encyclopædia Inflationaris
  source_content_hash: sha256:5c91b4e08970571a88bebe7f36c16c0a58ad55003328bece8a3f596484782565
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: cc-by-nc-sa
    url: https://github.com/cosmicinflation/aspic
  repo_url: https://github.com/cosmicinflation/aspic
  tools:
  - ASPIC
  - OpenMP
  - gfortran
  - libaspic
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# slow-roll-observable-prediction-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates reheating-consistent slow-roll observable predictions (spectral observables) for single-field inflationary models using the ASPIC/libaspic Fortran library, providing the forward model needed to compare theory with CMB data such as Planck likelihoods.

## When to use

Use when theoretical predictions of slow-roll single-field inflation models (spectral observables over a model prior/reheating grid) are needed as a forward model for CMB comparison or Bayesian model selection.

## When NOT to use

- The model of interest is not among the 74 inflationary models implemented in ASPIC and you do not intend to extend the code
- The task is non-slow-roll or multi-field inflation, outside the single-field slow-roll class ASPIC covers
- You only need parameter inference for a single already-fixed model rather than predictions across a model prior grid

## Inputs

- Selected single-field inflationary model(s) implemented in ASPIC (74 models supported)
- Values of the reheating energy density to scan over
- Built libaspic shared library (libaspic or quad-precision libaspicq)

## Outputs

- Reheating-consistent slow-roll spectral observables per model over the reheating/prior grid
- Per-model test program output from 'make check' / 'make test' runs

## How to apply

Build the shared scientific library libaspic from the cosmicinflation/aspic sources with gfortran using './configure', 'make', 'make install'; OpenMP parallelism is enabled by default via the '-fopenmp' flag. For each selected inflationary model (ASPIC presently supports 74), run the ASPIC slow-roll routines to compute reheating-consistent observable predictions for various values of the reheating energy density; the README suggests validating with 'make check' per model or the full 'make test' suite. If FORTRAN08 complex atan() calls error, recompile after 'export FCFLAGS="-DNOF08"'; if numerical precision limits are hit from extreme fine-tuning, rebuild with './configure --enable-quad-precision' (modules then in $PREFIX/include/aspicq, library renamed libaspicq). The resulting observable predictions serve as the forward model whose likelihood is evaluated against Planck CMB data in downstream Bayesian inference.

## Related tools

- **ASPIC** (Publicly available runtime library providing all routines needed to quickly derive reheating consistent observable predictions for slow-roll single field inflation models) — https://github.com/cosmicinflation/aspic
- **libaspic** (Shared scientific library of fast modern Fortran routines installed via './configure', 'make', 'make install'; documented via 'man libaspic') — https://github.com/cosmicinflation/aspic
- **gfortran** (Open source Fortran compiler supporting the FORTRAN08 complex atan() intrinsic calls used by ASPIC)
- **OpenMP** (Parallel processing of model predictions, switched by default using the '-fopenmp' flag; can be disabled with --disable-fopenmp)

## Examples

```
./configure --enable-quad-precision; make; make install; man libaspic
```

## Evaluation signals

- Library builds cleanly with './configure', 'make', 'make install' under gfortran with '-fopenmp' enabled
- 'make check' test programs in each model subdirectory terminate properly; the full 'make test' testsuite returns no error
- Generated predictions are reheating-consistent and cover various values of the reheating energy density
- Predicted observables are finite for each model on the prior/reheating grid (no precision-limitation error messages; otherwise switch to --enable-quad-precision)

## Limitations

- Some models require extreme fine-tuning of parameter values, causing numerical precision limitations that may require the slower quadruple-precision build (libaspicq)
- FORTRAN08 complex atan() support depends on recent compiler versions; older compilers need the FCFLAGS="-DNOF08" fallback
- ASPIC presently supports 74 models; unsupported models must be added by the user, and the code ships without version numbers or a changelog

## Evidence
