---
name: inflationary-potential-parameter-to-observable-mapping
description: Use when you need reheating-consistent n_s and r predictions from a specified
  single-field inflationary potential and its parameter values, e.g. for comparison
  with cosmological data or Bayesian model comparison.
when_to_use_negative:
- Your scenario is not a definite single-field inflationary model — ASPIC's libaspic
  routines compute observables only 'from definite single field inflationary models'
- You need predictions without specifying (or sweeping over) reheating conditions
  — libaspic predictions are explicitly reheating-consistent and require the reheating
  energy density as input
- You are not comparing against ASPIC reference values and only need generic numerical
  integration — the validation workflow is defined relative to libaspic output
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: ASPIC
  role: Publicly available runtime library providing all routines needed to derive
    reheating-consistent observable predictions for slow-roll single-field inflation
  repo: https://github.com/cosmicinflation/aspic
- name: libaspic
  role: Shared scientific library of fast modern Fortran routines whose per-model
    routines supply reference N, n_s, r values; documentation via 'man libaspic'
  repo: https://github.com/cosmicinflation/aspic
- name: libaspicq
  role: Quadruple-precision build of the library (modules in $PREFIX/include/aspicq)
    used when double-precision fine-tunings hit numerical precision limits
  repo: https://github.com/cosmicinflation/aspic
- name: gfortran
  role: Open-source Fortran compiler used to compile the library; recent versions
    support the FORTRAN08 complex atan() calls
- name: gcc
  role: Compatible compiler for the ASPIC build; OpenMP parallelism enabled via the
    '-fopenmp' flag
- name: Fortran
  role: Language in which the independent e-fold integral routine and libaspic routines
    are written and compared
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.dark.2014.01.003
    title: Encyclopædia Inflationaris
  source_content_hash: sha256:252912a3ed86f484adbd4d79b8a5eb3acff15acf294d6d06eae16d01e6ebc8e0
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
  - Fortran
  - gcc
  - gfortran
  - libaspic
  - libaspicq
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# inflationary-potential-parameter-to-observable-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates reheating-consistent slow-roll observable predictions (number of e-folds N, scalar spectral index n_s, tensor-to-scalar ratio r) for a definite single-field inflationary potential using the ASPIC libaspic Fortran runtime library, the codebase behind Encyclopædia Inflationaris.

## When to use

Use when you need reheating-consistent n_s and r predictions from a specified single-field inflationary potential and its parameter values, e.g. for comparison with cosmological data or Bayesian model comparison.

## When NOT to use

- Your scenario is not a definite single-field inflationary model — ASPIC's libaspic routines compute observables only 'from definite single field inflationary models'
- You need predictions without specifying (or sweeping over) reheating conditions — libaspic predictions are explicitly reheating-consistent and require the reheating energy density as input
- You are not comparing against ASPIC reference values and only need generic numerical integration — the validation workflow is defined relative to libaspic output

## Inputs

- Definition of a single-field inflationary potential and its parameter values (one of the ASPIC-supported models)
- Reheating energy density values over which predictions are swept
- Fortran model subdirectory under src/ with its test program

## Outputs

- Reheating-consistent slow-roll predictions: number of e-folds N, scalar spectral index n_s, tensor-to-scalar ratio r for each reheating energy density
- aspic_predictions.csv recording independently computed and libaspic reference values of N, n_s, r
- Test program output from each model subdirectory printing slow-roll predictions

## How to apply

Build and install the ASPIC runtime library from github.com/cosmicinflation/aspic with the standard './configure; make; make install' chain using gfortran; if FORTRAN08 complex 'atan()' intrinsic calls fail, re-export FCFLAGS="-DNOF08" before rebuilding. For the chosen model, use its subdirectory under src/, where a test program prints and generates slow-roll predictions for various values of the reheating energy density — the reheating density is the free input that ASPIC folds consistently into the e-fold integral and the horizon-exit evaluation of n_s and r. To validate, write an independent Fortran routine solving the slow-roll e-fold integral and comparing its N, n_s, r against the corresponding libaspic routine output within a tight numerical tolerance, recording all three observables in aspic_predictions.csv. If numerical precision errors prevent convergence due to extreme fine-tuning of parameter values, rebuild in quadruple precision with './configure --enable-quad-precision; make; make install', which relocates modules to $PREFIX/include/aspicq and renames the library libaspicq, then rerun the comparison. OpenMP is on by default via '-fopenmp'; disable it with '--disable-fopenmp' if the compiler is incompatible.

## Related tools

- **ASPIC** (Publicly available runtime library providing all routines needed to derive reheating-consistent observable predictions for slow-roll single-field inflation) — https://github.com/cosmicinflation/aspic
- **libaspic** (Shared scientific library of fast modern Fortran routines whose per-model routines supply reference N, n_s, r values; documentation via 'man libaspic') — https://github.com/cosmicinflation/aspic
- **libaspicq** (Quadruple-precision build of the library (modules in $PREFIX/include/aspicq) used when double-precision fine-tunings hit numerical precision limits) — https://github.com/cosmicinflation/aspic
- **gfortran** (Open-source Fortran compiler used to compile the library; recent versions support the FORTRAN08 complex atan() calls)
- **gcc** (Compatible compiler for the ASPIC build; OpenMP parallelism enabled via the '-fopenmp' flag)
- **Fortran** (Language in which the independent e-fold integral routine and libaspic routines are written and compared)

## Examples

```
export FCFLAGS="-DNOF08"; ./configure; make; make install
```

## Evaluation signals

- Independently computed N, n_s, and r match the libaspic reference values for the chosen model within a tight numerical tolerance
- All three observables (N, n_s, r) are present in aspic_predictions.csv for each reheating energy density tested
- The library's testsuite ('make test' / 'make check') terminates without error for the model's test program
- If the quadruple-precision libaspicq path was needed, the comparison still converges and records both precision paths' values
- Build artifacts match the expected layout: libaspic (or libaspicq) installed, modules in $PREFIX/include/aspic (or $PREFIX/include/aspicq)

## Limitations

- Some models require quite extreme fine-tunings of parameter values, causing numerical-precision errors that require the much slower quadruple-precision libaspicq build
- FORTRAN08 complex-number support for atan() intrinsic calls is only available in recent compilers; otherwise the -DNOF08 fallback must be applied
- OpenMP parallel processing (default '-fopenmp') may be incompatible with some compilers and must be disabled with --disable-fopenmp
- No version numbers for compilers or the library are specified in the documentation

## Evidence

- [abstract] The ASPIC code provides all routines needed to quickly derive reheating consistent observable predictions within this class of scenarios.: "The ASPIC code provides all routines needed to quickly derive reheating consistent observable predictions within this class of scenarios."
- [readme] In each model subdirectory (located under src/), a test program print and generate some slow-roll predictions for various values of the reheating energy density.: "In each model subdirectory (located under src/), a test program print and generate some slow-roll predictions for various values of the reheating energy density."
- [readme] If you get error messages due to numerical precision limitation, these may be overcome by compiling the library in quadruple precision (much slower): "If you get error messages due to numerical precision limitation, these may be overcome by compiling the library in quadruple precision (much slower)"
- [methods] export FCFLAGS="-DNOF08": "export FCFLAGS="-DNOF08""
