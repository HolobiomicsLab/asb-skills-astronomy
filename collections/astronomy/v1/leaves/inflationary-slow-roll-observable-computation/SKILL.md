---
name: inflationary-slow-roll-observable-computation
description: Use when you need reheating-consistent slow-roll predictions (n_s, r,
  number of e-folds) for one or more of the 74 single-field inflationary models supported
  by ASPIC, or when building libaspic from source.
when_to_use_negative:
- When the scenario is not slow-roll single-field inflation — ASPIC 'provides all
  routines needed to quickly derive reheating consistent observable predictions within
  this class of scenarios' only
- When the model of interest is not among the 74 supported ASPIC models and you are
  unwilling to implement and extend the library yourself
- When you need multi-field or non-slow-roll predictions, which the library does not
  target
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3298
tools:
- name: ASPIC
  role: Publicly available runtime library providing all routines needed to derive
    reheating-consistent observable predictions for slow-roll single-field inflation
    models
  repo: https://github.com/cosmicinflation/aspic
- name: libaspic
  role: Shared scientific library of fast modern Fortran routines installed by the
    build; invoked per model to obtain reference N, n_s and r values
  repo: https://github.com/cosmicinflation/aspic
- name: libaspicq
  role: Quadruple-precision variant of the library produced with --enable-quad-precision,
    used when numerical precision errors occur (much slower)
  repo: https://github.com/cosmicinflation/aspic
- name: gfortran
  role: Open-source Fortran compiler used to compile libaspic; supports the FORTRAN08
    complex atan() intrinsics unless FCFLAGS="-DNOF08" is set
- name: autoconf/configure build system
  role: Build system for configuring standard, quadruple-precision (--enable-quad-precision),
    or non-OpenMP (--disable-fopenmp) builds
  repo: https://github.com/cosmicinflation/aspic
- name: OpenMP
  role: Parallel processing switched on by default via the '-fopenmp' flag; deactivatable
    with --disable-fopenmp on incompatible compilers
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  source_papers:
  - doi: 10.1016/j.dark.2014.01.003
    title: Encyclopædia Inflationaris
  source_content_hash: sha256:26274c38f16d76c95b602732bb6f97bfff7c38e001446800828d9aa8f0c7b5e9
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
  - libaspicq
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3298
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# inflationary-slow-roll-observable-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute reheating-consistent slow-roll observable predictions (e.g. n_s and r as functions of reheating energy density) for single-field inflationary models using the ASPIC libaspic Fortran runtime library, either per-model via test programs or systematically across all 74 supported models.

## When to use

Use when you need reheating-consistent slow-roll predictions (n_s, r, number of e-folds) for one or more of the 74 single-field inflationary models supported by ASPIC, or when building libaspic from source.

## When NOT to use

- When the scenario is not slow-roll single-field inflation — ASPIC 'provides all routines needed to quickly derive reheating consistent observable predictions within this class of scenarios' only
- When the model of interest is not among the 74 supported ASPIC models and you are unwilling to implement and extend the library yourself
- When you need multi-field or non-slow-roll predictions, which the library does not target

## Inputs

- ASPIC source checkout from github.com/cosmicinflation/aspic (74 model subdirectories under src/)
- Choice of one inflationary potential model and its parameter set
- Grid of reheating energy density values
- Fortran compiler environment (gfortran/gcc, optional FCFLAGS such as -DNOF08)

## Outputs

- Per-model slow-roll predictions (n_s, r) for each reheating energy density grid point, e.g. slow_roll_observables.csv
- Optionally a comparison table aspic_predictions.csv with independently computed and libaspic-reference N, n_s, r
- Aggregated Encyclopædia Inflationaris table covering all 74 supported models

## How to apply

Obtain the source from github.com/cosmicinflation/aspic and build libaspic with the autoconf/configure chain: './configure', 'make', 'make install' with gfortran (OpenMP '-fopenmp' is on by default). If the atan() FORTRAN08 complex-number intrinsic calls fail, export FCFLAGS="-DNOF08" and rebuild; if a finely tuned model triggers numerical precision errors, rebuild in quadruple precision with './configure --enable-quad-precision', which installs modules in $PREFIX/include/aspicq as library 'libaspicq' (much slower); use './configure --disable-fopenmp' if '-fopenmp' is incompatible with your compiler. Then enter a chosen model subdirectory under src/ and run its test program (or 'make check' for per-model checks, 'make test' for the whole library) to print and generate slow-roll predictions for various values of the reheating energy density; sweep the reheating energy density over a grid and record predicted n_s and r per point (e.g. into slow_roll_observables.csv). Alternatively, write an independent Fortran routine that solves the slow-roll e-fold integral and evaluates the slow-roll parameters at horizon exit, then compare against libaspic reference values within a tight numerical tolerance. Reheating consistency is incorporated the way ASPIC does, treating the reheating energy density as the free parameter per prediction point.

## Related tools

- **ASPIC** (Publicly available runtime library providing all routines needed to derive reheating-consistent observable predictions for slow-roll single-field inflation models) — https://github.com/cosmicinflation/aspic
- **libaspic** (Shared scientific library of fast modern Fortran routines installed by the build; invoked per model to obtain reference N, n_s and r values) — https://github.com/cosmicinflation/aspic
- **libaspicq** (Quadruple-precision variant of the library produced with --enable-quad-precision, used when numerical precision errors occur (much slower)) — https://github.com/cosmicinflation/aspic
- **gfortran** (Open-source Fortran compiler used to compile libaspic; supports the FORTRAN08 complex atan() intrinsics unless FCFLAGS="-DNOF08" is set)
- **autoconf/configure build system** (Build system for configuring standard, quadruple-precision (--enable-quad-precision), or non-OpenMP (--disable-fopenmp) builds) — https://github.com/cosmicinflation/aspic
- **OpenMP** (Parallel processing switched on by default via the '-fopenmp' flag; deactivatable with --disable-fopenmp on incompatible compilers)

## Examples

```
./configure && make && make install; cd src/<model> && make check
```

## Evaluation signals

- The output table (e.g. slow_roll_observables.csv) contains numeric n_s and r entries for every reheating energy density grid point
- An aggregated Encyclopædia Inflationaris table covers all 74 supported models (row_count_equals 74) with a named observable field per model (field_present)
- Independently computed N, n_s and r match the libaspic reference values for the chosen model within numerical tolerance, with all three observables present in aspic_predictions.csv
- The test program runs to completion without numerical precision errors; 'make test' testsuite returns no error (it fails if any model test program fails to terminate properly)
- Build succeeds with default '-fopenmp'; if FCFLAGS="-DNOF08" or --enable-quad-precision was needed, the rebuilt library (libaspicq modules in $PREFIX/include/aspicq) still produces consistent predictions

## Limitations

- Some models require quite extreme fine-tunings; numerical precision limitations may force a quadruple-precision rebuild (--enable-quad-precision producing libaspicq), which is much slower
- atan() intrinsic calls use FORTRAN08 complex-number support and fail on older compilers, requiring export FCFLAGS="-DNOF08"
- The default '-fopenmp' parallel flag may be incompatible with some compilers, requiring --disable-fopenmp or a custom OpenMP flag via FCFLAGS
- Quadruple-precision builds are much slower; per-model 'make check' is slower than a plain build and whole-library 'make test' is much slower
- No version numbers for compilers or the library are specified ('recent versions of open source compilers such as gfortran')

## Evidence

- [abstract] ASPIC code provides all routines needed to quickly derive reheating consistent observable predictions: "The ASPIC code provides all routines needed to quickly derive reheating consistent observable predictions within this class of scenarios."
- [abstract] ASPIC presently supports 74 different inflationary models: "ASPIC has been designed as an evolutive code which presently supports 74 different models"
- [readme] In each model subdirectory (located under src/), a test program print and generate some slow-roll predictions: "In each model subdirectory (located under src/), a test program print and generate some slow-roll predictions for various values of the reheating energy density."
- [readme] If you ever encounter an error with these function calls try to define: export FCFLAGS="-DNOF08": "If you ever encounter an error with these function calls try to define:
bash
   export FCFLAGS="-DNOF08"
"
- [readme] compiling the library in quadruple precision (much slower) with ./configure --enable-quad-precision: "these may be overcome by compiling the library
in quadruple precision (much slower) with

bash
  ./configure --enable-quad-precision"
