---
name: reheating-consistent-prediction-sweep
description: Use when you need numerical slow-roll predictions (n_s, r) for one specific
  single-field inflationary model as a function of the reheating energy density, e.g.
  to fill model tables or feed Bayesian model comparison.
when_to_use_negative:
- You need multi-field inflation predictions or non-slow-roll dynamics — ASPIC is
  limited to reheating-consistent slow-roll single-field models
- You only need published tabulated results for all 74 models rather than a custom
  reheating grid — consult the Encyclopædia Inflationaris tables instead of rebuilding
- Your pipeline cannot handle a slow quadruple-precision build and the model requires
  extreme parameter fine-tuning that triggers numerical precision errors
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: ASPIC
  role: Runtime library providing routines to derive reheating-consistent observable
    predictions for slow-roll single-field inflation
  repo: https://github.com/cosmicinflation/aspic
- name: libaspic
  role: Shared scientific library of fast modern Fortran routines built and installed
    by the workflow; documented via 'man libaspic'
  repo: https://github.com/cosmicinflation/aspic
- name: gfortran
  role: Open-source Fortran compiler used to build libaspic; must support FORTRAN08
    complex atan() unless FCFLAGS="-DNOF08" is set
- name: OpenMP
  role: Parallel processing enabled by default via the '-fopenmp' flag; disable with
    --disable-fopenmp if the compiler is incompatible
- name: autoconf/configure build system
  role: Configures the build, including --enable-quad-precision and --disable-fopenmp
    variants
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.dark.2014.01.003
    title: Encyclopædia Inflationaris
  source_content_hash: sha256:a3c4298e826d8655c513fca9dd89bad7a20d3fe82523d79becf4fc4fa275ea22
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
  - autoconf/configure build system
  - gcc
  - gfortran
  - libaspic
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# reheating-consistent-prediction-sweep

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds the ASPIC/libaspic Fortran library and runs a single inflationary model's test program to sweep the reheating energy density, producing a table of reheating-consistent slow-roll observables (n_s, r). This is the per-model numerical workflow underlying the Encyclopædia Inflationaris.

## When to use

Use when you need numerical slow-roll predictions (n_s, r) for one specific single-field inflationary model as a function of the reheating energy density, e.g. to fill model tables or feed Bayesian model comparison.

## When NOT to use

- You need multi-field inflation predictions or non-slow-roll dynamics — ASPIC is limited to reheating-consistent slow-roll single-field models
- You only need published tabulated results for all 74 models rather than a custom reheating grid — consult the Encyclopædia Inflationaris tables instead of rebuilding
- Your pipeline cannot handle a slow quadruple-precision build and the model requires extreme parameter fine-tuning that triggers numerical precision errors

## Inputs

- ASPIC source checkout from github.com/cosmicinflation/aspic
- A selected single-field inflationary model subdirectory under src/ (one of 74 supported models)
- A grid of reheating energy density values
- Fortran compiler (gfortran/gcc) environment

## Outputs

- slow_roll_observables.csv with columns (reheating energy density, n_s, r) — one row per reheating grid point
- Per-model test program stdout printing slow-roll predictions for various reheating energy density values

## How to apply

Obtain a checkout of github.com/cosmicinflation/aspic and pick one of the 74 supported model subdirectories under src/. Build libaspic with './configure; make; make install'; if the Fortran compiler lacks FORTRAN08 complex-number support for the atan() intrinsic, export FCFLAGS="-DNOF08" before configuring; if numerical precision errors arise, rebuild in quadruple precision with './configure --enable-quad-precision' (producing libaspicq), and if '-fopenmp' is incompatible with the compiler, add './configure --disable-fopenmp'. Enter the chosen model directory and run its test program (or 'make check'), which prints and generates slow-roll predictions for various reheating energy density values. Sweep the reheating energy density over a grid and record n_s and r at each point. Validate that the output table has numeric n_s and r for every reheating grid point and that the program terminates without numerical precision errors.

## Related tools

- **ASPIC** (Runtime library providing routines to derive reheating-consistent observable predictions for slow-roll single-field inflation) — https://github.com/cosmicinflation/aspic
- **libaspic** (Shared scientific library of fast modern Fortran routines built and installed by the workflow; documented via 'man libaspic') — https://github.com/cosmicinflation/aspic
- **gfortran** (Open-source Fortran compiler used to build libaspic; must support FORTRAN08 complex atan() unless FCFLAGS="-DNOF08" is set)
- **OpenMP** (Parallel processing enabled by default via the '-fopenmp' flag; disable with --disable-fopenmp if the compiler is incompatible)
- **autoconf/configure build system** (Configures the build, including --enable-quad-precision and --disable-fopenmp variants)

## Examples

```
./configure && make && make install && cd src/<model> && make check
```

## Evaluation signals

- slow_roll_observables.csv exists and contains numeric n_s and r entries for every reheating energy density grid point
- The per-model test program runs to completion without numerical precision errors
- Build completes via './configure; make; make install' with no compiler errors (or with the documented FCFLAGS="-DNOF08" workaround applied)
- 'make check' in the model subdirectory terminates properly — the testsuite returns an error if a test program fails to terminate
- If quadruple precision was required, modules appear in $PREFIX/include/aspicq and the library is named libaspicq

## Limitations

- Some models require extreme fine-tuning of parameter values and may raise numerical precision errors, requiring the much slower quadruple-precision build (libaspicq)
- The atan() intrinsic calls rely on FORTRAN08 complex-number support, unavailable in older compilers; FCFLAGS="-DNOF08" is the fallback
- Parallel processing via the default '-fopenmp' flag may be incompatible with some compilers, requiring --disable-fopenmp
- Per-model checking via 'make check' is slower than the standard build, and 'make test' over the whole library is much slower
- No version numbers for compilers or the library are specified in the documentation

## Evidence
