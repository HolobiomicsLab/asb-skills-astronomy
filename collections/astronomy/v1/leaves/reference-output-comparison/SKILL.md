---
name: reference-output-comparison
description: Use when a locally built molscat, bound or field variant must be validated
  against the documentation's example outputs before production use, or when confirming
  documented values such as eigenphase-sum resonances or bound-state energies.
when_to_use_negative:
- When the model system or input parameters differ from those used to generate the
  reference output, since matched values are then not expected
- When optimisation level or BLAS/LAPACK implementations differ so substantially that
  bitwise reproducibility beyond numerical tolerance is demanded (README default is
  optimisation level 0 with traceback on)
- For reacting (reactive) collisions, since MOLSCAT/BOUND/FIELD are explicitly non-reactive
  quantum scattering and bound-state packages
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: molscat
  role: Executes the scattering calculation whose output is compared to the reference
    deposit
  repo: https://github.com/molscat/molscat
- name: bound
  role: Executes the bound-state calculation (e.g. bound-basic) whose eigenvalues
    are compared to the reference output
  repo: https://github.com/molscat/molscat
- name: GNUmakefile
  role: Builds the correct variant executable and sets Compiler, LIBS (optimised BLAS/LAPACK)
    or LIBUTILS (Netlib objects)
  repo: https://github.com/molscat/molscat
- name: Fortran
  role: Source language of the programs; compiler choice (gfortran/ifort) must match
    the build
  repo: https://github.com/molscat/molscat
- name: LAPACK
  role: Linear algebra routines (dsyevr, dsyr2k, dsytrf, etc.) underpinning the numerics
    being validated
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_001
  source_content_hash: sha256:823a7a4e2e9b63e5c3c24f6315cd34541d51155a843d68227a8e99b39c54a3f5
  source_papers:
  - doi: 10.48550/arxiv.1903.06755
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/molscat/molscat
  tools:
  - BLAS
  - Fortran
  - GNUmakefile
  - LAPACK
  - bound
  - gfortran
  - molscat
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# reference-output-comparison

## Summary

Reproduce a MOLSCAT/BOUND example calculation from its input file and diff the generated output against the deposited reference output in examples/output/ to verify that the locally built executable is numerically correct.

## When to use

Use when a locally built molscat, bound or field variant must be validated against the documentation's example outputs before production use, or when confirming documented values such as eigenphase-sum resonances or bound-state energies.

## When NOT to use

- When the model system or input parameters differ from those used to generate the reference output, since matched values are then not expected
- When optimisation level or BLAS/LAPACK implementations differ so substantially that bitwise reproducibility beyond numerical tolerance is demanded (README default is optimisation level 0 with traceback on)
- For reacting (reactive) collisions, since MOLSCAT/BOUND/FIELD are explicitly non-reactive quantum scattering and bound-state packages

## Inputs

- Namelist input file from examples/input/ (e.g. molscat-2chan-LJ.input, bound-2chan-LJ.input, molscat-basic1.input)
- Reference output file from examples/output/ (e.g. bound-2chan-LJ.output)
- Locally built executable (molscat-, bound- or field- variant) compiled via GNUmakefile

## Outputs

- Generated program output file matching the reference deposit to printed precision
- Verification record (e.g. bound-2chan-LJ-verification.txt) with extracted energies or cross sections
- Extracted quantities: eigenphase sums, bound-state energies (cm-1), or integral cross sections SIG(j,j') (A^2)

## How to apply

First adapt the GNUmakefile for the working environment (set Compiler to gfortran, put optimised BLAS/LAPACK in LIBS or Netlib object files in LIBUTILS), then build the target variant with a GNU make rule such as 'make molscat-H2_H2'. Run the executable with the corresponding namelist input file (&INPUT, &BASIS, &POTL blocks read on unit 5) from examples/input/ supplied on standard input, performing the same close-coupling or bound-state calculation specified there. Extract the key quantities from the produced output — e.g. eigenphase sums near 42.84 cm-1 for the Feshbach-resonance scan, bound-state eigenvalues near -58.2 and -12.1 cm-1, or state-to-state integral cross sections SIG(1,1)=1.81057 A^2 and SIG(2,1)=3.722327E-02 A^2 — and compare each against the reference output in examples/output/ to the printed precision. Acceptance is met only when every compared value matches within numerical tolerance.

## Related tools

- **molscat** (Executes the scattering calculation whose output is compared to the reference deposit) — https://github.com/molscat/molscat
- **bound** (Executes the bound-state calculation (e.g. bound-basic) whose eigenvalues are compared to the reference output) — https://github.com/molscat/molscat
- **GNUmakefile** (Builds the correct variant executable and sets Compiler, LIBS (optimised BLAS/LAPACK) or LIBUTILS (Netlib objects)) — https://github.com/molscat/molscat
- **Fortran** (Source language of the programs; compiler choice (gfortran/ifort) must match the build) — https://github.com/molscat/molscat
- **LAPACK** (Linear algebra routines (dsyevr, dsyr2k, dsytrf, etc.) underpinning the numerics being validated)

## Examples

```
make molscat-H2_H2 && ./molscat-H2_H2 < examples/input/molscat-2chan-LJ.input > molscat-2chan-LJ.output
```

## Evaluation signals

- Generated output file is identical to, or numerically identical to printed precision with, the reference file in examples/output/ (acceptance criterion cited in section 14.1.12)
- Eigenphase-sum scan for molscat-2chan-LJ.input shows the Feshbach resonance feature located just below 42.84 cm-1
- bound-2chan-LJ.input yields the two converged bound states at approximately -58.2 and -12.1 cm-1
- molscat-basic1.output cross sections match section 3.8.1 values (SIG(1,1)=1.81057 A^2, SIG(2,1)=3.722327E-02 A^2, inelastic totals 3.72239E-02, 8.77864E-03, 7.47803E-05 A^2)
- Make rule such as 'make molscat-H2_H2' completes and the executable writes output when fed the namelist input on unit 5

## Limitations

- GNUmakefile must first be adapted (LIBS/LIBUTILS, Compiler); defaults assume gfortran at optimisation level 0 with traceback on
- If GDATE/GTIME/GCLOCK intrinsics (date_and_time, cpu_time) are unavailable they must be replaced locally; GDATE/GTIME may return blanks and GCLOCK 0.D0
- Optimised BLAS/LAPACK in LIBS vs Netlib object files in LIBUTILS can change low-order digits, so comparison must be within numerical tolerance, not bitwise
- If EXECDIR/OBJDIR are changed from '.', the commented-out $(PROGS) rule line must be uncommented and directories created, otherwise make warns about circular dependency

## Evidence
