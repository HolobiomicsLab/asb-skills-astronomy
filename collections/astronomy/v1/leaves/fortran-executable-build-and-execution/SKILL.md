---
name: fortran-executable-build-and-execution
description: Use when a task requires compiling a variant of MOLSCAT/BOUND/FIELD from
  the molscat/molscat source tree and executing it on an example .input file to generate
  or verify quantitative outputs such as bound-state energies in cm-1 or Feshbach
  resonance fields in Gauss.
when_to_use_negative:
- The goal is reactive quantum scattering — MOLSCAT handles non-reactive collisions
  only
- A pre-built executable and reference output already exist and only the scientific
  results are needed, not the build/run workflow
- The analysis is post-processing of existing S-matrix files (differential cross sections,
  RESFIT resonance fitting) rather than running the main programs
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_3332
tools:
- name: molscat 2025.0
  role: Built as variant executables (e.g. molscat-Rb2) for scattering-length scans
    over magnetic field
  repo: https://github.com/molscat/molscat
- name: bound 2025.0
  role: Built as variant executables (e.g. bound-basic) to converge bound-state energies
    from namelist input
  repo: https://github.com/molscat/molscat
- name: field 2025.0
  role: Built as variant executables (e.g. field-Rb2) to search over magnetic field
    for threshold crossings
  repo: https://github.com/molscat/molscat
- name: GNUmakefile (GNU make)
  role: Drives compilation; LIBS/LIBUTILS variables must be adapted to local BLAS/LAPACK
    availability
  repo: https://github.com/molscat/molscat
- name: gfortran
  role: Default Fortran compiler set in the Compiler variable of GNUmakefile
- name: LAPACK
  role: Linear algebra routines the executables link against via LIBS or LIBUTILS
- name: BLAS
  role: Basic linear algebra routines required by the LAPACK calls in the executables
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_002
  - task_005
  source_content_hash: sha256:8cdc677e0a1296b475de5bde1129aefcb4b5c6d579ebdffc37cd1ca4cada2171
  source_papers:
  - doi: 10.48550/arxiv.1903.06755
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/molscat/molscat
  tools:
  - BLAS
  - Fortran
  - GNUmakefile (GNU make)
  - LAPACK
  - bound 2025.0
  - field 2025.0
  - gfortran
  - molscat
  - molscat 2025.0
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# fortran-executable-build-and-execution

## Summary

Build and run the MOLSCAT/BOUND/FIELD Fortran 90 executables (e.g. bound-basic, molscat-Rb2, field-Rb2) by adapting GNUmakefile LIBS/LIBUTILS for BLAS/LAPACK, then executing a namelist input file on unit 5 to reproduce published bound-state energies, threshold-crossing fields, or scattering-length scans.

## When to use

Use when a task requires compiling a variant of MOLSCAT/BOUND/FIELD from the molscat/molscat source tree and executing it on an example .input file to generate or verify quantitative outputs such as bound-state energies in cm-1 or Feshbach resonance fields in Gauss.

## When NOT to use

- The goal is reactive quantum scattering — MOLSCAT handles non-reactive collisions only
- A pre-built executable and reference output already exist and only the scientific results are needed, not the build/run workflow
- The analysis is post-processing of existing S-matrix files (differential cross sections, RESFIT resonance fitting) rather than running the main programs

## Inputs

- Fortran 90 source files (.f / .f90) in source_code/ and the GNUmakefile
- Namelist input file on unit 5 with &INPUT, &BASIS and &POTL blocks (e.g. examples/input/bound-2chan-LJ.input, molscat-basic_Rb2.input)
- Optimised BLAS/LAPACK libraries (LIBS) or Netlib object files (LIBUTILS)
- Potential data files from the data/ directory (e.g. Mg + NH, H2 + H2)

## Outputs

- Executable in EXECDIR (e.g. bound-basic, molscat-Rb2, field-Rb2) with object files in OBJDIR
- Program output with converged bound-state energies (e.g. -58.2 and -12.1 cm-1), threshold-crossing fields (e.g. 851.8761789 G), or field-dependent scattering lengths
- Verification/comparison file (e.g. bound-2chan-LJ-verification.txt) or CSV table (e.g. rb2_scattering_length_vs_field.csv)

## How to apply

Adapt GNUmakefile for the local environment: set LIBS to optimised BLAS/LAPACK libraries (daxpy, dcopy, dgemm, dgemv, dgesv, dscal, dswap, dsymm, idamax; dgesv, dlamch, dsyevr, dsyr2k, dsytrf, dsytri, ilaenv, lsame) or place Netlib object files in LIBUTILS. Build the desired variant with a GNU make rule following the prefix convention (e.g. 'make bound-basic' or 'make field-Rb2'), noting the compiler defaults to gfortran and optimisation level 0 with traceback on. Supply the namelist input file (&INPUT, &BASIS, &POTL blocks) on unit 5 (standard input), e.g. bound-2chan-LJ.input, bound-basic1.input, molscat-basic_Rb2.input or field-basic_Rb2.input. Extract converged results from the output — bound-state eigenvalues, 'CONVERGED ON STATE NUMBER n' lines, or per-field scattering lengths — and compare against the reference files in examples/output/ or documented values (e.g. sections 3.9, 3.10.2 and 14.3.2 of the manual). Record verification in a comparison file such as bound-2chan-LJ-verification.txt.

## Related tools

- **molscat 2025.0** (Built as variant executables (e.g. molscat-Rb2) for scattering-length scans over magnetic field) — https://github.com/molscat/molscat
- **bound 2025.0** (Built as variant executables (e.g. bound-basic) to converge bound-state energies from namelist input) — https://github.com/molscat/molscat
- **field 2025.0** (Built as variant executables (e.g. field-Rb2) to search over magnetic field for threshold crossings) — https://github.com/molscat/molscat
- **GNUmakefile (GNU make)** (Drives compilation; LIBS/LIBUTILS variables must be adapted to local BLAS/LAPACK availability) — https://github.com/molscat/molscat
- **gfortran** (Default Fortran compiler set in the Compiler variable of GNUmakefile)
- **LAPACK** (Linear algebra routines the executables link against via LIBS or LIBUTILS)
- **BLAS** (Basic linear algebra routines required by the LAPACK calls in the executables)

## Examples

```
make bound-basic && ./bound-basic < examples/input/bound-2chan-LJ.input > bound-2chan-LJ.output
```

## Evaluation signals

- Run output matches the reference deposit in examples/output/ (e.g. bound-2chan-LJ.output) within numerical tolerance
- BOUND converges on the documented states: e.g. states 4 and 5 at -4.992074672 and -1.413103892 cm-1 for bound-basic1.input
- FIELD reports 'CONVERGED ON STATE NUMBER n' at fields matching 851.8761789, 819.5625939 and 810.9471815 Gauss for 85Rb2
- Scattering-length scan completes without error for all 101 field points (800–900 G in 1 G steps) and shows a pole within a few gauss of 850 G
- Executables appear in EXECDIR and object files in OBJDIR as configured in GNUmakefile

## Limitations

- GNUmakefile must be adapted to the local environment; optimised BLAS/LAPACK must be supplied in LIBS or Netlib objects in LIBUTILS
- Compiler is gfortran by default and must be changed if unavailable; optimisation is 0 with traceback on until the code is validated
- GDATE/GTIME/GCLOCK use f90 intrinsics (date_and_time, cpu_time) that may need local replacement if unavailable
- If EXECDIR is changed from the current directory, the commented dependency line in GNUmakefile must be uncommented and the directory must already exist

## Evidence
