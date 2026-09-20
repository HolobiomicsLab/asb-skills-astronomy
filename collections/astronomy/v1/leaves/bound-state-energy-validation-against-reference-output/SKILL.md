---
name: bound-state-energy-validation-against-reference-output
description: Use when a BOUND run on an atom–rigid-rotor system must be validated
  against documented reference energies, e.g. after rebuilding an executable or changing
  BLAS/LAPACK libraries.
when_to_use_negative:
- Input is a scattering (molscat) run rather than a bound-state (bound) calculation
  — this skill validates bound-state boundary-condition outputs only.
- The energy window or JTOT/IBFIX settings differ from the bound-basic1 example, so
  the section 3.9 reference energies (-4.992074672 and -1.413103892 cm-1) no longer
  apply as validation targets.
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3332
tools:
- name: bound
  role: Executes the coupled-channel bound-state calculation for the atom–rigid-rotor
    system under the bound-basic1 namelist settings
  repo: https://github.com/molscat/molscat
- name: GNU make / GNUmakefile
  role: Builds the bound-basic executable after adapting LIBS/LIBUTILS for the local
    BLAS/LAPACK environment
- name: LAPACK
  role: Linear algebra routines used by bound wherever possible during the coupled-channel
    solve
- name: BLAS
  role: Basic linear algebra subroutines used by the LAPACK routines in the bound-state
    solve
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:4a942bbb7bfbeaed886bf5e2bf184b1f1baa726ada902465c7b9bcfdcab99a06
  source_papers:
  - doi: 10.48550/arxiv.1903.06755
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/molscat/molscat
  tools:
  - BLAS
  - BOUND
  - GNU make / GNUmakefile
  - LAPACK
  - bound
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# bound-state-energy-validation-against-reference-output

## Summary

Reproduce and validate converged bound-state energies from the BOUND example bound-basic1 (JTOT=1, IBFIX=2, EMIN=-10 to EMAX=-1 cm-1) by comparing computed state energies against the section 3.9 documented values (-4.992074672 cm-1 for state 4 and -1.413103892 cm-1 for state 5).

## When to use

Use when a BOUND run on an atom–rigid-rotor system must be validated against documented reference energies, e.g. after rebuilding an executable or changing BLAS/LAPACK libraries.

## When NOT to use

- Input is a scattering (molscat) run rather than a bound-state (bound) calculation — this skill validates bound-state boundary-condition outputs only.
- The energy window or JTOT/IBFIX settings differ from the bound-basic1 example, so the section 3.9 reference energies (-4.992074672 and -1.413103892 cm-1) no longer apply as validation targets.

## Inputs

- bound-basic1.input namelist input file (from examples/input/) specifying EMIN=-10 cm-1, EMAX=-1 cm-1, JTOT=1, IBFIX=2
- built bound-basic executable linked against BLAS/LAPACK (LIBS) or Netlib routines (LIBUTILS)

## Outputs

- bound-basic1.output file containing converged bound-state energies for JTOT=1, IBFIX=2
- validated energy values for state 4 (-4.992074672 cm-1) and state 5 (-1.413103892 cm-1)

## How to apply

Locate bound-basic1.input in examples/input/ of the molscat distribution and run the bound-basic executable with that input on unit 5; the namelist blocks set EMIN=-10 cm-1, EMAX=-1 cm-1, JTOT=1 and IBFIX=2. Build the executable first with GNU make, adapting the GNUmakefile LIBS variable to point at optimised BLAS/LAPACK libraries (or Netlib routines in LIBUTILS). Inspect the program output for the converged bound-state energies at JTOT=1 with IBFIX=2 and identify the energies for state number 4 and state number 5. Judge correctness by matching the computed energies to the section 3.9 documented values: -4.992074672 cm-1 for state 4 and -1.413103892 cm-1 for state 5, to full printed precision. Save the run output as bound-basic1.output in the manner of the examples/output/ files for later diffing.

## Related tools

- **bound** (Executes the coupled-channel bound-state calculation for the atom–rigid-rotor system under the bound-basic1 namelist settings) — https://github.com/molscat/molscat
- **GNU make / GNUmakefile** (Builds the bound-basic executable after adapting LIBS/LIBUTILS for the local BLAS/LAPACK environment)
- **LAPACK** (Linear algebra routines used by bound wherever possible during the coupled-channel solve)
- **BLAS** (Basic linear algebra subroutines used by the LAPACK routines in the bound-state solve)

## Examples

```
make bound-basic && ./bound-basic < examples/input/bound-basic1.input > bound-basic1.output
```

## Evaluation signals

- Converged energy for state 4 equals -4.992074672 cm-1 exactly to printed precision
- Converged energy for state 5 equals -1.413103892 cm-1 exactly to printed precision
- Output was produced for the run with JTOT=1, IBFIX=2 and EMIN=-10.0 to EMAX=-1.0 cm-1 as in the input file
- Run output saved as bound-basic1.output and diffs cleanly against the corresponding file in examples/output/
- Build succeeded with the adapted GNUmakefile (LIBS pointing at optimised BLAS/LAPACK or LIBUTILS holding Netlib routines)

## Limitations

- Results depend on the BLAS/LAPACK libraries linked at build time; the README warns that optimised versions or Netlib routines must be correctly configured in LIBS/LIBUTILS.
- Compiler settings default to gfortran at optimisation level 0 with traceback switched on; changing COMPILE.f after 'assuring yourself that the code works' may alter run behaviour.
- Validation targets are specific to the bound-basic1 example run (section 3.9); other examples (sections 3.7, 3.8, chapter 13) have different reference outputs.

## Evidence

- [results] For this run, bound converges on state number 4 at energy -4.992074672 cm-1 and state number 5 at energy -1.413103892 cm-1.: [evidence span withheld — anchor status: not_found]
- [abstract] BOUND is a general-purpose package for performing calculations of bound-state energies in weakly bound atomic and molecular systems using coupled-channel methods.: "BOUND is a general-purpose package for performing calculations of bound-state energies in weakly bound atomic and molecular systems using coupled-channel methods"
- [readme] All the input data files used for example calculations in chapter 3 ... The input files are contained in the directory examples/input/ and the output files are contained in the directory examples/output/.: "The input files are contained in the directory examples/input/ and the output files are contained in the directory examples/output/"
- [readme] GNUmakefile first needs to be adapted for your own working environment. MOLSCAT, BOUND and FIELD executables make use of BLAS and LAPACK library routines.: "MOLSCAT, BOUND and FIELD executables make use of BLAS and LAPACK library routines.  If optimised versions of these library routines are available, include the libraries containing them in the"
- [abstract] The programs use LAPACK linear algebra routines wherever possible.: "The programs use LAPACK linear algebra routines wherever possible"
