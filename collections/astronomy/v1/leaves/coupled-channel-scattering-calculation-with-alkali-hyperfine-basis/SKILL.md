---
name: coupled-channel-scattering-calculation-with-alkali-hyperfine-basis
description: Use when you need field-dependent s-wave scattering lengths or resonance
  locations for two alkali-metal atoms at ultracold collision energies, from coupled-channel
  calculations with hyperfine structure in a magnetic field.
when_to_use_negative:
- The collision system is not two alkali-metal atoms with hyperfine structure in a
  magnetic field — the alkali-metal plug-in basis-set suite does not cover other interacting
  species
- The calculation is a bound-state or Feshbach-resonance-location problem rather than
  a scattering-length scan — BOUND or FIELD are the appropriate programs, not a molscat
  field scan
- Collisions are reactive or involve molecule-molecule interactions not covered by
  the built-in coupled-equation generators or available plug-in routines
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3450
- http://edamontology.org/topic_3318
tools:
- name: molscat 2025.0
  role: Coupled-channel non-reactive quantum scattering program; the molscat-Rb2 variant
    computes s-wave scattering lengths and effective ranges as a function of magnetic
    field
  repo: https://github.com/molscat/molscat
- name: GNUmakefile
  role: Build file adapted for the working environment (LIBS/LIBUTILS variables) to
    compile the molscat-Rb2 executable with optimised BLAS/LAPACK
  repo: https://github.com/molscat/molscat
- name: BLAS
  role: Linear algebra kernels (dgemm, dsymm, etc.) linked into the executable for
    the coupled-channel propagator
- name: LAPACK
  role: Linear algebra routines (dsyevr, dsytrf, etc.) used by the programs wherever
    possible
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:99853b2ee5b56ab49ee0270ef499b03b38e9128c9c3237d010fdbd25b008c0df
  source_papers:
  - doi: 10.48550/arxiv.1903.06755
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/molscat/molscat
  tools:
  - BLAS
  - GNUmakefile
  - LAPACK
  - molscat
  - molscat 2025.0
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3450
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# coupled-channel-scattering-calculation-with-alkali-hyperfine-basis

## Summary

Builds and runs a MOLSCAT 2025.0 variant with the alkali-metal atom + alkali-metal atom basis-set suite (including hyperfine structure in an external magnetic field) to compute low-energy scattering lengths and effective ranges as a function of magnetic field, e.g. for the Rb-85 + Rb-85 Feshbach-resonance scan.

## When to use

Use when you need field-dependent s-wave scattering lengths or resonance locations for two alkali-metal atoms at ultracold collision energies, from coupled-channel calculations with hyperfine structure in a magnetic field.

## When NOT to use

- The collision system is not two alkali-metal atoms with hyperfine structure in a magnetic field — the alkali-metal plug-in basis-set suite does not cover other interacting species
- The calculation is a bound-state or Feshbach-resonance-location problem rather than a scattering-length scan — BOUND or FIELD are the appropriate programs, not a molscat field scan
- Collisions are reactive or involve molecule-molecule interactions not covered by the built-in coupled-equation generators or available plug-in routines

## Inputs

- molscat namelist input file (e.g. molscat-basic_Rb2.input) with &INPUT, &BASIS and &POTL blocks specifying interaction type, basis set, potential, reduced mass, propagator, collision energy (100 nK) and magnetic-field scan (800–900 G in 1 G steps)
- MOLSCAT 2025.0 Fortran 90 source code (source_code directory) and GNUmakefile
- Optimised BLAS/LAPACK libraries or Netlib fallback routines

## Outputs

- molscat-Rb2 executable built from the alkali-metal atom + alkali-metal atom variant source
- Per-field-point scattering output listing the calculated s-wave scattering length (and effective range) for each of the 101 magnetic-field values
- Table of field (G) vs scattering length, e.g. rb2_scattering_length_vs_field.csv

## How to apply

Adapt GNUmakefile for your environment, placing optimised BLAS (daxpy, dcopy, dgemm, dgemv, dgesv, dscal, dswap, dsymm, idamax) and LAPACK (dgesv, dlamch, dsyevr, dsyr2k, dsytrf, dsytri, ilaenv, lsame) routines in the LIBS variable, then build the molscat-Rb2 variant of MOLSCAT 2025.0. Run molscat-Rb2 on a namelist input file (e.g. molscat-basic_Rb2.input) that sets up the alkali-metal atom + alkali-metal atom basis with hyperfine structure via the &INPUT, &BASIS and &POTL blocks, scanning the magnetic field from 800 to 900 G in 1 G steps at a collision energy of 100 nK. MOLSCAT performs close-coupling calculations with no dynamical approximations, looping over total angular momentum (partial wave), and for low-energy scattering reports the s-wave scattering length and effective range at each field point. Collect the per-field scattering-length values into a table (field in G vs scattering length, e.g. rb2_scattering_length_vs_field.csv) and inspect the curve: a wide resonance appears as a pole near 850 G and a narrow resonance near 820 G.

## Related tools

- **molscat 2025.0** (Coupled-channel non-reactive quantum scattering program; the molscat-Rb2 variant computes s-wave scattering lengths and effective ranges as a function of magnetic field) — https://github.com/molscat/molscat
- **GNUmakefile** (Build file adapted for the working environment (LIBS/LIBUTILS variables) to compile the molscat-Rb2 executable with optimised BLAS/LAPACK) — https://github.com/molscat/molscat
- **BLAS** (Linear algebra kernels (dgemm, dsymm, etc.) linked into the executable for the coupled-channel propagator)
- **LAPACK** (Linear algebra routines (dsyevr, dsytrf, etc.) used by the programs wherever possible)

## Examples

```
make molscat-Rb2 && ./molscat-Rb2 < molscat-basic_Rb2.input > molscat-basic_Rb2.output
```

## Evaluation signals

- The run completes without error for all 101 field points (800–900 G in 1 G intervals)
- The computed scattering-length curve contains a pole within a few gauss of 850 G
- A narrow resonance feature appears within a few gauss of 820 G in the scattering-length vs field plot
- MOLSCAT reports both a scattering length and an effective range for low-energy scattering at each field value
- The CSV table (rb2_scattering_length_vs_field.csv) has one scattering-length entry per 1 G field step, matching section 3.10.1 behaviour

## Limitations

- Optimisation level is set to 0 with traceback on by default in COMPILE.f; increase optimisation only after verifying the code works
- If optimised BLAS/LAPACK are unavailable, the requisite routines must be downloaded from Netlib and included via LIBUTILS, and many LAPACK routines call other BLAS/LAPACK routines
- GDATE, GTIME and GCLOCK rely on f90 intrinsics (date_and_time, cpu_time); if unavailable they must be replaced with local routines
- The GNUmakefile assumes GNU make and gfortran by default; other compilers require editing the Compiler variable
- GDATE and GTIME should return blank character variables in the last resort, and GCLOCK a double precision value of 0.D0

## Evidence

- [readme] For low-energy scattering, MOLSCAT can calculate scattering lengths and effective ranges and can locate and characterize scattering resonances as a function of an external variable such as the magnetic field.: "For low-energy scattering, MOLSCAT can calculate scattering lengths and effective ranges and can locate and characterize scattering resonances"
