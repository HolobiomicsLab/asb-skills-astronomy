---
name: quantum-bound-state-output-interpretation
description: Use when a BOUND (or bound-basic) run has finished and you need to identify
  converged bound-state eigenvalues in the program output and verify them against
  a reference output file.
when_to_use_negative:
- The calculation is a scattering run with MOLSCAT (S matrices, cross sections, scattering
  lengths) rather than a bound-state energy calculation
- The goal is to locate field values at which a bound state exists at a specified
  energy — use FIELD instead of interpreting plain BOUND output
edam_operation: http://edamontology.org/operation_2479
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: BOUND
  role: BOUND / bound-basic executable that performs the coupled-channel bound-state
    calculation whose output eigenvalues are interpreted and validated
  repo: https://github.com/molscat/molscat
- name: LAPACK
  role: Linear algebra routines used by the BOUND executable to solve the coupled-channel
    eigenvalue problem
- name: BLAS
  role: Basic linear algebra subroutines required when linking the bound-basic executable
    via LIBS or LIBUTILS
- name: Fortran
  role: Source language of BOUND; built with gfortran via GNUmakefile
provenance:
  source_task_ids:
  - task_004
  source_content_hash: sha256:18137b38af0e1bbbd28bcb5e343598ae94f42c5478cf96e36a9d4f6875c3e879
  source_papers:
  - doi: 10.48550/arxiv.1903.06755
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/molscat/molscat
  tools:
  - BLAS
  - BOUND
  - Fortran
  - LAPACK
  - gfortran
  edam_operation: http://edamontology.org/operation_2479
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# quantum-bound-state-output-interpretation

## Summary

Interprets BOUND program output for coupled-channel bound-state calculations, extracting converged bound-state eigenvalues (in cm-1) and validating them against reference deposit outputs, as demonstrated for the two-channel Lennard-Jones test problem.

## When to use

Use when a BOUND (or bound-basic) run has finished and you need to identify converged bound-state eigenvalues in the program output and verify them against a reference output file.

## When NOT to use

- The calculation is a scattering run with MOLSCAT (S matrices, cross sections, scattering lengths) rather than a bound-state energy calculation
- The goal is to locate field values at which a bound state exists at a specified energy — use FIELD instead of interpreting plain BOUND output

## Inputs

- BOUND namelist input file (e.g. examples/input/bound-2chan-LJ.input with &INPUT, &BASIS, &POTL blocks)
- Compiled bound-basic executable built from source_code/ via GNUmakefile
- Reference output file (e.g. examples/output/bound-2chan-LJ.output) for validation

## Outputs

- Converged bound-state eigenvalues in cm-1 extracted from the run output (e.g. -58.2 and -12.1 cm-1)
- Verification record file (e.g. bound-2chan-LJ-verification.txt) confirming agreement with the reference

## How to apply

Build the bound-basic executable from the Fortran 90 source in source_code/ using GNUmakefile, linking optimised BLAS/LAPACK via LIBS or Netlib object files via LIBUTILS. Run bound-basic with the input file (e.g. examples/input/bound-2chan-LJ.input) supplied on standard input; the input is read on unit 5 and consists of namelist blocks &INPUT, &BASIS and &POTL specifying interaction type, basis set, potential, reduced mass and propagator. Extract the converged bound-state eigenvalues from the program output (for the two-channel Lennard-Jones test these are approximately -58.2 and -12.1 cm-1, weakly perturbed from the single-channel levels at -57.7 and -11.9 cm-1). Compare the computed energies against the reference file examples/output/bound-2chan-LJ.output within numerical tolerance, and record the check in a verification file such as bound-2chan-LJ-verification.txt.

## Related tools

- **BOUND** (BOUND / bound-basic executable that performs the coupled-channel bound-state calculation whose output eigenvalues are interpreted and validated) — https://github.com/molscat/molscat
- **LAPACK** (Linear algebra routines used by the BOUND executable to solve the coupled-channel eigenvalue problem)
- **BLAS** (Basic linear algebra subroutines required when linking the bound-basic executable via LIBS or LIBUTILS)
- **Fortran** (Source language of BOUND; built with gfortran via GNUmakefile)

## Examples

```
./bound-basic < examples/input/bound-2chan-LJ.input > bound-2chan-LJ-run.output
```

## Evaluation signals

- The run output matches the reference deposit file (e.g. bound-2chan-LJ.output) with both bound-state energies reproduced within numerical tolerance
- For the two-channel Lennard-Jones test, two bound states are located at approximately -58.2 and -12.1 cm-1
- The computed two-channel levels are only weakly perturbed from the corresponding single-channel levels (-57.7 and -11.9 cm-1), consistent with expected channel coupling magnitude
- The check is recorded in a verification file such as bound-2chan-LJ-verification.txt

## Limitations

- Interpretation is specific to bound-state (BOUND) output; MOLSCAT and FIELD variants have different outputs and use cases
- The executable must be built with BLAS/LAPACK available (optimised libraries in LIBS or Netlib objects in LIBUTILS), otherwise the build/run will not work
- GNUmakefile defaults to gfortran at optimisation level 0 with traceback on; changing compilers or optimisation levels requires editing COMPILE.f and the Compiler variable

## Evidence

- [results] Running bound on the two-channel Lennard-Jones model (bound-2chan-LJ.input) locates two bound states, at -58.2 cm-1 and -12.1 cm-1, weakly perturbed from the corresponding single-channel levels at -57.7 and -11.9 cm-1.: [evidence span withheld — anchor status: not_found]
- [other] Validation: the run output matches the reference deposit bound-2chan-LJ.output, with both bound-state energies reproduced within numerical tolerance.: [evidence span withheld — anchor status: not_found]
