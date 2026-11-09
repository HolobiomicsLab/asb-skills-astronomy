---
name: anisotropic-potential-expansion-interpretation
description: Use when a molscat/BOUND run specifies an anisotropic potential via &POTL
  (MXLAM, LAMBDA, NTERM, NPOWER, A) and you need to reproduce or replace it with a
  user-supplied VSTAR routine, or verify the printed POTENL expansion matches documented
  coefficients.
when_to_use_negative:
- When the interaction potential is a simple one already fully expressible as input
  data (the README states 'Simple interaction potentials may be specified in input
  data and more complicated ones may be handled with plug-in routines')
- When the collision type or external-field treatment is the part being customised
  — that is handled by basis-set-suite plug-ins, not the VSTAR potential hook
- When the potential is tabulated data for Mg + NH or H2 + H2 supplied in the data/
  directory rather than an analytic expansion
tools:
- name: molscat
  role: Scattering program whose POTENL routine routes negative-NTERM expansion terms
    through the VSTAR plug-in; its printed POTENL block validates the expansion
  repo: https://github.com/molscat/molscat
- name: Fortran
  role: Language for writing the VINIT/VSTAR plug-in routine returning radial coefficients
    P(I)
  repo: https://github.com/molscat/molscat
- name: LAPACK
  role: Linear algebra routines linked when building the molscat executable used to
    test the VSTAR potential
  repo: https://github.com/molscat/molscat
- name: BLAS
  role: Basic linear algebra subroutines required by LAPACK in the GNUmakefile LIBS/LIBUTILS
    build of molscat
  repo: https://github.com/molscat/molscat
provenance:
  source_task_ids:
  - task_008
  source_content_hash: sha256:c1d07b3634e13298d9b152b23d48d86e2d5419e55086b78cc7c4f79edf9a468b
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# anisotropic-potential-expansion-interpretation

## Summary

Interprets and re-implements the anisotropic interaction-potential expansion read from the molscat/BOUND &POTL namelist (MXLAM, LAMBDA, NTERM, NPOWER, A), including routing selected expansion terms to a user-supplied VINIT/VSTAR plug-in routine instead of direct input coefficients. This lets the same model potential (e.g. two lambda=0 terms plus one lambda=2 term) be supplied either as data or programmatically.

## When to use

Use when a molscat/BOUND run specifies an anisotropic potential via &POTL (MXLAM, LAMBDA, NTERM, NPOWER, A) and you need to reproduce or replace it with a user-supplied VSTAR routine, or verify the printed POTENL expansion matches documented coefficients.

## When NOT to use

- When the interaction potential is a simple one already fully expressible as input data (the README states 'Simple interaction potentials may be specified in input data and more complicated ones may be handled with plug-in routines')
- When the collision type or external-field treatment is the part being customised — that is handled by basis-set-suite plug-ins, not the VSTAR potential hook
- When the potential is tabulated data for Mg + NH or H2 + H2 supplied in the data/ directory rather than an analytic expansion

## Inputs

- molscat input file with &POTL namelist block (MXLAM, LAMBDA, NTERM, NPOWER, A arrays)
- Fortran 90 VINIT/VSTAR user potential routine
- molscat .f/.f90 source files and GNUmakefile from the molscat/molscat repository

## Outputs

- molscat run whose printed POTENL expansion lists MXLAM=2, LAMBDA=0,2, NTERM=2,1, NPOWER=-12,-6,-6, A=1.0,-2.0,-0.2
- VSTAR-supplied radial coefficients P(I) as a function of distance R for each expansion term
- S-matrix / cross-section output equivalent to the data-supplied potential run

## How to apply

First parse the &POTL namelist to recover the expansion: for the model system MXLAM=2, LAMBDA=0,2, NTERM=2,1, NPOWER=-12,-6,-6 and A=1.0,-2.0,-0.2, giving lambda=0 terms 1.0*R^-12 - 2.0*R^-6 and a lambda=2 term -0.2*R^-6. In the general-purpose POTENL routine, any term whose NTERM element is negative is not read as data: the code sets P(I)=0.D0 and executes IF (NTERM(I).LT.0) CALL VSTAR(R, I, P(I)), so the plug-in flags its terms with negative NTERM values and VSTAR returns the radial coefficient in P(I) for each term index I. Write a VINIT/VSTAR Fortran 90 routine that returns 1.0*R^-12 - 2.0*R^-6 for term I=1 (lambda=0) and -0.2*R^-6 for term I=2 (lambda=2), build the executable with gfortran via GNUmakefile (including BLAS/LAPACK: daxpy, dcopy, dgemm, dgemv, dgesv, dscal, dswap, dsymm, idamax plus dlamch, dsyevr, dsyr2k, dsytrf, dsytri, ilaenv, lsame), link the routine, and run with examples/input/molscat-basic1.input. Rationale: negative NTERM is the documented hook for delegating complicated potentials to plug-in routines rather than encoding them in the input file.

## Related tools

- **molscat** (Scattering program whose POTENL routine routes negative-NTERM expansion terms through the VSTAR plug-in; its printed POTENL block validates the expansion) — https://github.com/molscat/molscat
- **Fortran** (Language for writing the VINIT/VSTAR plug-in routine returning radial coefficients P(I)) — https://github.com/molscat/molscat
- **LAPACK** (Linear algebra routines linked when building the molscat executable used to test the VSTAR potential) — https://github.com/molscat/molscat
- **BLAS** (Basic linear algebra subroutines required by LAPACK in the GNUmakefile LIBS/LIBUTILS build of molscat) — https://github.com/molscat/molscat

## Examples

```
make molscat-basic1 && ./molscat-basic1 < examples/input/molscat-basic1.input
```

## Evaluation signals

- The POTENL printout lists MXLAM=2 with LAMBDA=0,2, NTERM=2,1, NPOWER=-12,-6,-6 and A=1.0,-2.0,-0.2 exactly as documented in section 3.8.1
- VSTAR returns P(1) = 1.0*R^-12 - 2.0*R^-6 for the lambda=0 term and P(2) = -0.2*R^-6 for the lambda=2 term at spot-check distances R
- Scattering output (S matrices / cross sections) from the VSTAR-supplied run matches the run with the same potential given directly via &POTL data
- Every term intended for VSTAR appears in the input with a negative NTERM element; no data coefficient is silently ignored
- The build succeeds with the required BLAS/LAPACK routines (dgemm, dsyevr, dlamch, etc.) linked per GNUmakefile

## Limitations

- Only expansion terms flagged with negative NTERM are routed to VSTAR; ordinary terms are still taken from the A coefficients in the input file
- GNUmakefile must be adapted for the working environment (LIBS or LIBUTILS for BLAS/LAPACK), and the compiler default gfortran may need changing
- Optimisation level is 0 with traceback on by default; performance changes only after the code is verified

## Evidence

- [full_text] the code sets P(I) = 0.D0 and then, for such terms, executes IF (NTERM(I).LT.0) CALL VSTAR(R, I, P(I)): [evidence span withheld — anchor status: not_found]
- [readme] Simple interaction potentials may be specified in input data and more complicated ones may be handled with plug-in routines.: "Simple interaction potentials may be specified in input data and more complicated ones may be handled with plug-in routines."
- [abstract] namelist input blocks &INPUT, &BASIS, &POTL: [evidence span withheld — anchor status: not_found]
- [readme] GNUmakefile first needs to be adapted for your own working environment: "GNUmakefile first needs to be adapted for your own working environment.  MOLSCAT, BOUND and FIELD executables make use of BLAS and LAPACK library routines."
