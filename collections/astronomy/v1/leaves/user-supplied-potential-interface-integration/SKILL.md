---
name: user-supplied-potential-interface-integration
description: Use when a molscat/BOUND/FIELD potential cannot be expressed as namelist
  NPOWER/A terms in &POTL and must instead be evaluated by a Fortran routine via the
  VSTAR(R, I, P(I)) interface.
when_to_use_negative:
- When the potential is fully expressible as simple NPOWER/A power-law terms in the
  &POTL namelist — plain input data is the intended route for 'simple interaction
  potentials'.
- When the collision type or basis set (rather than the potential) is the non-standard
  element — use the plug-in basis-set suite interface instead.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: molscat 2025.0
  role: scattering program whose general-purpose POTENL calls the user-supplied VSTAR
    routine for negative-NTERM potential terms
  repo: https://github.com/molscat/molscat
- name: Fortran
  role: language in which the VINIT/VSTAR plug-in potential routine is written (near-standard
    Fortran 77 with Fortran 90 features)
  repo: https://github.com/molscat/molscat
- name: LAPACK
  role: linear algebra library linked into the executable being built with the plug-in
    routine
  repo: https://github.com/molscat/molscat
- name: BLAS
  role: basic linear algebra subroutines required by LAPACK when compiling the executable
  repo: https://github.com/molscat/molscat
provenance:
  source_task_ids:
  - task_008
  source_content_hash: sha256:c32700acc7749d957d2ed3afd837ad7236420691849cb42ed307fcff13620b2f
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
  - molscat 2025.0
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# user-supplied-potential-interface-integration

## Summary

Implement a user-supplied VINIT/VSTAR potential plug-in routine that molscat's general-purpose POTENL calls for any expansion term flagged with a negative NTERM value, replacing namelist-supplied radial coefficients with code-computed ones.

## When to use

Use when a molscat/BOUND/FIELD potential cannot be expressed as namelist NPOWER/A terms in &POTL and must instead be evaluated by a Fortran routine via the VSTAR(R, I, P(I)) interface.

## When NOT to use

- When the potential is fully expressible as simple NPOWER/A power-law terms in the &POTL namelist — plain input data is the intended route for 'simple interaction potentials'.
- When the collision type or basis set (rather than the potential) is the non-standard element — use the plug-in basis-set suite interface instead.

## Inputs

- Fortran 90 source files (.f/.f90) of the molscat code base in source_code/
- GNUmakefile adapted for the working environment (LIBS / LIBUTILS variables)
- User-written VINIT/VSTAR potential routine returning radial coefficients P(I) for each flagged term index I
- molscat input file (e.g. examples/input/molscat-basic1.input) with &POTL namelist using negative NTERM entries, MXLAM=2, LAMBDA=0,2

## Outputs

- molscat executable with the plug-in potential routine linked
- molscat run output whose printed POTENL expansion lists the lambda values and radial coefficients supplied by VSTAR

## How to apply

Build the base molscat executable from the .f/.f90 sources in source_code with gfortran via GNUmakefile, placing BLAS (daxpy, dcopy, dgemm, dgemv, dgesv, dscal, dswap, dsymm, idamax) and LAPACK routines in LIBS or LIBUTILS. In the &POTL namelist, declare each potential term with a negative NTERM entry (with its LAMBDA value listed via LAMBDA/MXLAM); POTENL then sets P(I)=0.D0 and executes IF (NTERM(I).LT.0) CALL VSTAR(R, I, P(I)), so your routine receives the distance R and term index I and returns the radial coefficient in P(I). For the model system of chapter 3.8.1, write VSTAR so term I=1 (lambda=0) returns 1.0*R^-12 - 2.0*R^-6 and term I=2 (lambda=2) returns -0.2*R^-6, matching MXLAM=2, LAMBDA=0,2, NTERM=2,1, NPOWER=-12,-6,-6 and A=1.0,-2.0,-0.2. Optionally implement VINIT for one-time initialisation before the first VSTAR call. Run with the examples/input/ file molscat-basic1.input linked to your routine and compare the printed POTENL expansion against the documented section 3.8.1 values.

## Related tools

- **molscat 2025.0** (scattering program whose general-purpose POTENL calls the user-supplied VSTAR routine for negative-NTERM potential terms) — https://github.com/molscat/molscat
- **Fortran** (language in which the VINIT/VSTAR plug-in potential routine is written (near-standard Fortran 77 with Fortran 90 features)) — https://github.com/molscat/molscat
- **LAPACK** (linear algebra library linked into the executable being built with the plug-in routine) — https://github.com/molscat/molscat
- **BLAS** (basic linear algebra subroutines required by LAPACK when compiling the executable) — https://github.com/molscat/molscat

## Examples

```
make molscat-H2_H2  # then run: ./molscat-basic1 < examples/input/molscat-basic1.input with the VINIT/VSTAR routine linked
```

## Evaluation signals

- The POTENL printout lists MXLAM=2 with LAMBDA=0,2, NTERM=2,1, NPOWER=-12,-6,-6 and A=1.0,-2.0,-0.2 exactly as documented in section 3.8.1.
- VSTAR is invoked with the correct term index I: I=1 returns 1.0*R^-12 - 2.0*R^-6 (lambda=0) and I=2 returns -0.2*R^-6 (lambda=2).
- Each user-supplied term is flagged by a negative NTERM entry; POTENL sets P(I)=0.D0 before the VSTAR call, so no hardcoded namelist A value should appear for those terms.
- The executable builds cleanly with gfortran via GNUmakefile with the listed BLAS and LAPACK routines present in LIBS or LIBUTILS.
- Scattering results from the VSTAR route reproduce the run with the equivalent namelist-supplied potential (molscat-basic1.input values).

## Limitations

- VSTAR is reached only through the general-purpose POTENL path for terms flagged with negative NTERM; potentials handled by special-purpose executables (e.g. molscat-Mg_NH, molscat-H2_H2 built from data/ files) use different routines.
- The plug-in must be written in the code's Fortran dialect (Fortran 90 with legacy Fortran 77 features) and recompile the executable; it cannot be injected at runtime.
- GNUmakefile defaults to gfortran at optimisation level 0 with traceback on; compiler or BLAS/LAPACK availability issues require editing LIBS, LIBUTILS, or the Compiler variable.

## Evidence

- [other] Within the general-purpose POTENL, any potential expansion term whose NTERM element is negative is obtained from the user-supplied routine: [evidence span withheld — anchor status: not_found]
- [other] IF (NTERM(I).LT.0) CALL VSTAR(R, I, P(I)): [evidence span withheld — anchor status: not_found]
