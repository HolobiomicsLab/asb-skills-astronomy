---
name: fortran-potential-routine-development
description: Use when a molscat/BOUND/FIELD &POTL potential cannot be expressed as
  input-data power terms (NPOWER, A) and must instead come from a user-coded routine,
  or when reproducing a documented model potential (e.g. molscat-basic1) via the plug-in
  route.
when_to_use_negative:
- The potential can be fully expressed as simple input-data power terms — use the
  &POTL namelist route (positive NTERM with NPOWER/A) instead of a plug-in.
- The interaction requires a plug-in basis-set suite or external-field coupling (magnetic,
  electric, photon fields) rather than just a radial potential routine.
- A reactive collision type is needed — molscat only handles non-reactive quantum
  scattering with its coded/internal and plug-in atom-molecule and molecule-molecule
  types.
edam_operation: http://edamontology.org/operation_2406
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: molscat
  role: Target program into which the VINIT/VSTAR potential plug-in is linked; its
    POTENL routine calls VSTAR(R, I, P(I)) for negative-NTERM terms and prints the
    resulting expansion for validation
  repo: https://github.com/molscat/molscat
- name: Fortran
  role: Language in which the VINIT/VSTAR routine is written; sources compiled with
    gfortran (default) via GNUmakefile
  repo: https://github.com/molscat/molscat
- name: LAPACK
  role: Linear algebra library required at build time (dgesv, dlamch, dsyevr, dsyr2k,
    dsytrf, dsytri, ilaenv, lsame) for the executables using the plug-in
- name: BLAS
  role: Basic linear algebra routines (daxpy, dcopy, dgemm, dgemv, dgesv, dscal, dswap,
    dsymm, idamax) required at build time via LIBS or LIBUTILS
provenance:
  source_task_ids:
  - task_008
  source_content_hash: sha256:43d37771328a6a7824ccf38ca8a47d47cd9d5e4b70cfe37f70181ce0da134c94
  source_papers:
  - doi: 10.48550/arxiv.1903.06755
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/molscat/molscat
  tools:
  - BLAS
  - Fortran
  - LAPACK
  - molscat
  edam_operation: http://edamontology.org/operation_2406
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# fortran-potential-routine-development

## Summary

Develop a user-supplied VINIT/VSTAR plug-in potential routine in Fortran 90 for molscat, replacing namelist-specified radial expansion terms (NPOWER/A coefficients in &POTL) with code that returns the lambda-resolved radial coefficients V_lambda(R) at runtime.

## When to use

Use when a molscat/BOUND/FIELD &POTL potential cannot be expressed as input-data power terms (NPOWER, A) and must instead come from a user-coded routine, or when reproducing a documented model potential (e.g. molscat-basic1) via the plug-in route.

## When NOT to use

- The potential can be fully expressed as simple input-data power terms — use the &POTL namelist route (positive NTERM with NPOWER/A) instead of a plug-in.
- The interaction requires a plug-in basis-set suite or external-field coupling (magnetic, electric, photon fields) rather than just a radial potential routine.
- A reactive collision type is needed — molscat only handles non-reactive quantum scattering with its coded/internal and plug-in atom-molecule and molecule-molecule types.

## Inputs

- molscat/BOUND/FIELD Fortran source files (.f/.f90) from source_code/ and the GNUmakefile
- Potential expansion specification: MXLAM, LAMBDA, NTERM, NPOWER and A values from the &POTL namelist (e.g. molscat-basic1.input)
- Chapter 5 documentation describing the VINIT/VSTAR plug-in interface
- BLAS/LAPACK libraries (optimised versions or Netlib object code)

## Outputs

- A Fortran 90 VINIT/VSTAR routine returning radial potential coefficients P(I) for each negative-NTERM expansion term
- A linked molscat variant executable with the user potential routine
- A run output whose POTENL printout reproduces the documented expansion (MXLAM=2, LAMBDA=0,2, NTERM=2,1, NPOWER=-12,-6,-6, A=1.0,-2.0,-0.2)

## How to apply

Build the base molscat executable from the .f/.f90 sources with gfortran via GNUmakefile, setting LIBS or LIBUTILS to point at the required BLAS (daxpy, dcopy, dgemm, dgemv, dgesv, dscal, dswap, dsymm, idamax) and LAPACK routines (dgesv, dlamch, dsyevr, dsyr2k, dsytrf, dsytri, ilaenv, lsame). Write a Fortran 90 VINIT/VSTAR routine that implements VSTAR(R, I, P(I)), the interface the general-purpose POTENL calls for any expansion term whose NTERM element is negative: VSTAR receives the distance R and term index I and must return the radial coefficient in P(I). Flag the terms with negative NTERM values — for the molscat-basic1 model system, VSTAR returns 1.0*R^-12 - 2.0*R^-6 for term I=1 (lambda=0) and -0.2*R^-6 for term I=2 (lambda=2), matching the &POTL namelist values MXLAM=2, LAMBDA=0,2, NTERM=2,1, NPOWER=-12,-6,-6, A=1.0,-2.0,-0.2. Link the routine into a variant executable and run it with examples/input/ molscat-basic1.input; validation requires the printed POTENL expansion to list exactly those LAMBDA, NTERM, NPOWER and A values as documented in section 3.8.1.

## Related tools

- **molscat** (Target program into which the VINIT/VSTAR potential plug-in is linked; its POTENL routine calls VSTAR(R, I, P(I)) for negative-NTERM terms and prints the resulting expansion for validation) — https://github.com/molscat/molscat
- **Fortran** (Language in which the VINIT/VSTAR routine is written; sources compiled with gfortran (default) via GNUmakefile) — https://github.com/molscat/molscat
- **LAPACK** (Linear algebra library required at build time (dgesv, dlamch, dsyevr, dsyr2k, dsytrf, dsytri, ilaenv, lsame) for the executables using the plug-in)
- **BLAS** (Basic linear algebra routines (daxpy, dcopy, dgemm, dgemv, dgesv, dscal, dswap, dsymm, idamax) required at build time via LIBS or LIBUTILS)

## Examples

```
make molscat-basic1 && ./molscat-basic1 < examples/input/molscat-basic1.input
```

## Evaluation signals

- The run's POTENL printout lists MXLAM=2 with LAMBDA=0,2, NTERM=2,1, NPOWER=-12,-6,-6 and A=1.0,-2.0,-0.2 exactly as documented in section 3.8.1
- VSTAR is only invoked for terms flagged with negative NTERM values, for which POTENL sets P(I) = 0.D0 before the call
- The program compiles and links cleanly with the adapted GNUmakefile (LIBS/LIBUTILS resolving all BLAS/LAPACK symbols) and default gfortran, optimisation level 0 with traceback on
- Output of the plug-in run matches the output of the equivalent namelist-based run (molscat-basic1) for the same collision system

## Limitations

- Only expansion terms with negative NTERM elements are routed to VSTAR; terms given as positive NTERM must still be supplied via NPOWER/A input data
- The VSTAR interface passes only the distance R and the term index I, so the routine must internally hold any potential parameters and lambda associations
- The code is near-standard Fortran 77 with some Fortran 90 features; GNUmakefile sets optimisation level 0 with traceback on and expects GDATE/GTIME/GCLOCK to be available (or replaced by local routines)

## Evidence

- [other] VSTAR is called with the distance R and the term index I and returns the radial potential coefficient in P(I): [evidence span withheld — anchor status: not_found]
- [other] the code sets P(I) = 0.D0 and then, for such terms, executes IF (NTERM(I).LT.0) CALL VSTAR(R, I, P(I)): [evidence span withheld — anchor status: not_found]
