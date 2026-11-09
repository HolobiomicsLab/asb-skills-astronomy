---
name: reheating-consistent-parameter-inference
description: Use when you need slow-roll inflationary predictions for a definite single-field
  model that consistently marginalize over the unknown reheating energy density, e.g.
  to feed a CMB likelihood or Bayesian model comparison.
when_to_use_negative:
- The model of interest is not among ASPIC's supported single-field slow-roll models
  and you cannot add it to the code yourself
- You need predictions that fix the reheating history to a single known value rather
  than marginalizing over the reheating energy density
- Your analysis is non-slow-roll or multi-field, which is outside the 'definite single
  field inflationary models' scope of ASPIC
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: ASPIC
  role: Publicly available runtime library providing all routines needed to derive
    reheating-consistent observable predictions for slow-roll single-field inflation
    models
  repo: https://github.com/cosmicinflation/aspic
- name: libaspic
  role: Shared scientific library of fast Fortran routines installed from cosmicinflation/aspic
    via ./configure && make && make install; link against it to compute model observables
  repo: https://github.com/cosmicinflation/aspic
- name: gfortran
  role: Open-source Fortran compiler used to build libaspic, including FORTRAN08 complex
    atan() support needed by some model routines
- name: OpenMP
  role: Parallel processing enabled by default via the '-fopenmp' flag to speed up
    evaluations over model prior grids
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.dark.2014.01.003
    title: Encyclopædia Inflationaris
  source_content_hash: sha256:294bd441bde6dbb57bb99f662c8c0a19cddab8bf63cd5ad402c8c784cf138971
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
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# reheating-consistent-parameter-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute reheating-consistent observable predictions (e.g. spectral index, tensor-to-scalar ratio) for single-field slow-roll inflation models using the ASPIC/libaspic Fortran library, so that model parameters can be constrained or models compared against cosmological data such as Planck CMB observables.

## When to use

Use when you need slow-roll inflationary predictions for a definite single-field model that consistently marginalize over the unknown reheating energy density, e.g. to feed a CMB likelihood or Bayesian model comparison.

## When NOT to use

- The model of interest is not among ASPIC's supported single-field slow-roll models and you cannot add it to the code yourself
- You need predictions that fix the reheating history to a single known value rather than marginalizing over the reheating energy density
- Your analysis is non-slow-roll or multi-field, which is outside the 'definite single field inflationary models' scope of ASPIC

## Inputs

- Definition of a single-field inflationary model (one of ASPIC's 74 supported models)
- Model prior parameter grid over which observables are evaluated
- Reheating energy density values (predictions are generated for various reheating energy densities)
- Cosmological data for comparison (e.g. Planck CMB likelihood on spectral observables)

## Outputs

- Reheating-consistent slow-roll predictions of spectral observables per model (e.g. n_s, r) on the prior grid
- Parameter constraints / likelihood values when predictions are compared to CMB data
- Bayesian evidence values and model rankings when used for model comparison (e.g. written to bayesian_evidence_ranking.csv)

## How to apply

Build the shared library libaspic from the cosmicinflation/aspic sources with './configure', 'make', 'make install' (gfortran, OpenMP on by default via '-fopenmp'). Select the target inflationary model from ASPIC's supported set (74 models) and run its slow-roll routines to generate predictions for the spectral observables over the model's prior grid, treating the reheating energy density consistently. If the compiler errors on FORTRAN08 complex 'atan()' calls, retry with 'export FCFLAGS="-DNOF08"'; if numerical precision limits are hit for fine-tuned models, rebuild with './configure --enable-quad-precision' (modules then land in $PREFIX/include/aspicq as libaspicq). Evaluate the predictions against CMB data (e.g. Planck likelihoods) to constrain parameters, or feed the forward model into a Bayesian evidence calculation with an Occam penalty from the prior volume. Per-model sanity checks can be run with 'make check' (or 'make test' for the whole testsuite), which executes test programs printing slow-roll predictions for various reheating energy densities.

## Related tools

- **ASPIC** (Publicly available runtime library providing all routines needed to derive reheating-consistent observable predictions for slow-roll single-field inflation models) — https://github.com/cosmicinflation/aspic
- **libaspic** (Shared scientific library of fast Fortran routines installed from cosmicinflation/aspic via ./configure && make && make install; link against it to compute model observables) — https://github.com/cosmicinflation/aspic
- **gfortran** (Open-source Fortran compiler used to build libaspic, including FORTRAN08 complex atan() support needed by some model routines)
- **OpenMP** (Parallel processing enabled by default via the '-fopenmp' flag to speed up evaluations over model prior grids)

## Examples

```
export FCFLAGS="-DNOF08"; ./configure && make && make install && man libaspic
```

## Evaluation signals

- The built library installs and 'man libaspic' documentation is accessible
- 'make check' (or 'make test') per-model test programs terminate properly for each model, printing slow-roll predictions for various reheating energy densities
- Generated spectral observables are finite across the model prior grid and consistent with reheating-consistency constraints
- When used for Bayesian comparison, the evidence ranking table (e.g. bayesian_evidence_ranking.csv) has one row per selected model with finite evidences ordered monotonically by evidence
- Quad-precision rebuild (libaspicq) resolves numerical precision errors reported for strongly fine-tuned models

## Limitations

- Some models require extreme fine-tuning of parameter values and can hit numerical precision limits; quadruple precision (libaspicq) is much slower and modules move to $PREFIX/include/aspicq
- FORTRAN08 complex atan() calls may fail on older compilers; the '-DNOF08' fallback disables that support
- No version numbers for compilers or the library are documented, and OpenMP may be incompatible with some proprietary compilers (use --disable-fopenmp)
- ASPIC currently supports 74 models; unlisted models require the user to add support themselves

## Evidence
