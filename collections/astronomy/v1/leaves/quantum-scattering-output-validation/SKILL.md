---
name: quantum-scattering-output-validation
description: Use when a MOLSCAT calculation delegates potential terms to a user VSTAR
  routine via negative NTERM entries, and you must verify that the printed POTENL
  expansion matches the expected radial coefficients before accepting the run.
when_to_use_negative:
- Potential terms are supplied entirely by input data (positive NTERM/NPOWER/A in
  &POTL) with no VSTAR plug-in — there is no user routine to validate
- The calculation uses a special-purpose potential data file variant (e.g. molscat-Mg_NH
  or molscat-H2_H2 executables with data/ files) rather than the general-purpose POTENL
  with user VSTAR terms
- The goal is validating scattering observables (S matrices, cross sections, scattering
  lengths) rather than the potential expansion itself
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3332
tools:
- name: molscat 2025.0
  role: Executes the coupled-channel scattering run and prints the POTENL expansion
    used for validation
  repo: https://github.com/molscat/molscat
- name: Fortran
  role: Language for writing the VINIT/VSTAR plug-in routine whose coefficients are
    validated
  repo: https://github.com/molscat/molscat
- name: LAPACK
  role: Linear algebra routines required to build the MOLSCAT executable under validation
  repo: https://github.com/molscat/molscat
- name: BLAS
  role: Basic linear algebra subroutines required to build the MOLSCAT executable
    under validation
  repo: https://github.com/molscat/molscat
provenance:
  source_task_ids:
  - task_008
  source_content_hash: sha256:a8e093c09f24f9c24a21eebb57733a85949176efbaabab922b7846909451c074
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# quantum-scattering-output-validation

## Summary

Validation skill for MOLSCAT runs that supply potential expansion terms through a user VINIT/VSTAR plug-in: inspect the printed POTENL expansion in the program output and confirm the LAMBDA/NTERM/NPOWER/A values reproduce the documented model-system coefficients (e.g. molscat-basic1: LAMBDA=0,2; NTERM=2,1; NPOWER=-12,-6,-6; A=1.0,-2.0,-0.2).

## When to use

Use when a MOLSCAT calculation delegates potential terms to a user VSTAR routine via negative NTERM entries, and you must verify that the printed POTENL expansion matches the expected radial coefficients before accepting the run.

## When NOT to use

- Potential terms are supplied entirely by input data (positive NTERM/NPOWER/A in &POTL) with no VSTAR plug-in — there is no user routine to validate
- The calculation uses a special-purpose potential data file variant (e.g. molscat-Mg_NH or molscat-H2_H2 executables with data/ files) rather than the general-purpose POTENL with user VSTAR terms
- The goal is validating scattering observables (S matrices, cross sections, scattering lengths) rather than the potential expansion itself

## Inputs

- MOLSCAT namelist input file (e.g. examples/input/molscat-basic1.input with &POTL: MXLAM=2, LAMBDA=0,2, NTERM=2,1, NPOWER=-12,-6,-6, A=1.0,-2.0,-0.2)
- User-supplied VINIT/VSTAR Fortran 90 potential routine returning 1.0*R^-12 - 2.0*R^-6 for term I=1 (lambda=0) and -0.2*R^-6 for term I=2 (lambda=2)
- Program output file from the MOLSCAT run containing the printed POTENL expansion

## Outputs

- Verified POTENL expansion printout matching the documented section 3.8.1 values
- Pass/fail validation decision for the VINIT/VSTAR plug-in run

## How to apply

Build the base MOLSCAT executable from the .f/.f90 sources with gfortran via GNUmakefile, including the required BLAS and LAPACK routines. Run molscat with the examples/input/ file molscat-basic1.input (MXLAM=2, LAMBDA=0,2, NTERM=2,1, NPOWER=-12,-6,-6, A=1.0,-2.0,-0.2) linked against the user VINIT/VSTAR routine, which flags terms with negative NTERM so POTENL calls VSTAR(R, I, P(I)) to fill each radial coefficient. Inspect the program output and confirm the printed POTENL expansion matches the documented section 3.8.1 values. The run is accepted only if the POTENL printout lists MXLAM=2 with LAMBDA=0,2, NTERM=2,1, NPOWER=-12,-6,-6 and A=1.0,-2.0,-0.2 exactly as documented; any mismatch indicates the VSTAR routine returns the wrong coefficient for a given term index I.

## Related tools

- **molscat 2025.0** (Executes the coupled-channel scattering run and prints the POTENL expansion used for validation) — https://github.com/molscat/molscat
- **Fortran** (Language for writing the VINIT/VSTAR plug-in routine whose coefficients are validated) — https://github.com/molscat/molscat
- **LAPACK** (Linear algebra routines required to build the MOLSCAT executable under validation) — https://github.com/molscat/molscat
- **BLAS** (Basic linear algebra subroutines required to build the MOLSCAT executable under validation) — https://github.com/molscat/molscat

## Examples

```
make molscat-basic1 && ./molscat-basic1 < examples/input/molscat-basic1.input > molscat-basic1.out  # then check POTENL printout for LAMBDA=0,2; NTERM=2,1; NPOWER=-12,-6,-6; A=1.0,-2.0,-0.2
```

## Evaluation signals

- POTENL printout lists MXLAM=2 with LAMBDA=0,2
- POTENL printout lists NTERM=2,1 and NPOWER=-12,-6,-6
- POTENL printout lists A=1.0,-2.0,-0.2 exactly as documented in section 3.8.1
- Term I=1 (lambda=0) evaluates to 1.0*R^-12 - 2.0*R^-6 and term I=2 (lambda=2) evaluates to -0.2*R^-6 via VSTAR
- Negative NTERM flags in &POTL correctly trigger VSTAR calls rather than input-data coefficients

## Limitations

- Validation covers only the printed potential expansion; it does not verify convergence of propagator settings or cross sections
- Negative NTERM semantics exist only within the general-purpose POTENL; special-purpose variants built with potential data files (e.g. Mg + NH, H2 + H2) follow different construction paths
- GNUmakefile must first be adapted for the working environment (LIBS or LIBUTILS for BLAS/LAPACK); a misbuilt executable invalidates the comparison

## Evidence

- [full_text] the code sets P(I) = 0.D0 and then, for such terms, executes IF (NTERM(I).LT.0) CALL VSTAR(R, I, P(I)): [evidence span withheld — anchor status: not_found]
- [full_text] any potential expansion term whose NTERM element is negative is obtained from the user-supplied routine rather than input data: [evidence span withheld — anchor status: not_found]
