---
name: namelist-input-file-configuration-for-bound
description: Use when you need to set up or reproduce a BOUND bound-state energy calculation
  (e.g. bound-basic) for an atom–rigid-rotor system and must specify the energy window
  (EMIN/EMAX), total angular momentum (JTOT), and symmetry/fixing options (IBFIX)
  in namelist input blocks.
when_to_use_negative:
- The goal is scattering observables (S matrices, cross sections, scattering lengths,
  resonances) rather than bound-state energies — that requires MOLSCAT, not BOUND
  namelist configuration
- The task is locating external-field values at which a bound state exists at a specified
  energy (e.g. Feshbach resonance positions) — that requires FIELD, a development
  of BOUND
- The coupled equations are not among the built-in basis types (e.g. atom + linear
  rigid rotor) and no plug-in basis-set suite is available
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3318
tools:
- name: bound 2025.0
  role: Executable that reads the namelist input on unit 5 and performs coupled-channel
    bound-state energy calculations for weakly bound atomic and molecular systems
  repo: https://github.com/molscat/molscat
- name: GNUmakefile / make
  role: Build system used to compile the bound-basic variant; LIBS/LIBUTILS variables
    must be adapted to available BLAS/LAPACK libraries before running
  repo: https://github.com/molscat/molscat
- name: LAPACK
  role: Linear algebra routines used by BOUND for the coupled-channel eigenvalue solves
    underlying the bound-state energies
  repo: https://github.com/molscat/molscat
- name: BLAS
  role: Basic linear algebra subroutines used by the LAPACK routines in BOUND; optimised
    versions are linked via the LIBS variable
  repo: https://github.com/molscat/molscat
- name: Fortran
  role: Language of the namelist input format and the BOUND source; the input file
    consists of Fortran namelist data blocks read on unit 5
  repo: https://github.com/molscat/molscat
provenance:
  source_task_ids:
  - task_002
  source_content_hash: sha256:9a1094ffbb949a1e3542e6ee44baec8a49eeae436a2903fc82c0e507fdae9fad
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
  - GNUmakefile / make
  - LAPACK
  - bound 2025.0
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# namelist-input-file-configuration-for-bound

## Summary

Configure the Fortran namelist input file (&INPUT, &BASIS, &POTL blocks read on unit 5) that drives a BOUND bound-state calculation for a weakly bound atom–molecule system, so that the bound-basic executable converges on the correct bound-state energies. This is the standard way to reproduce documented BOUND example runs such as the JTOT=1, IBFIX=2 atom–rigid-rotor case.

## When to use

Use when you need to set up or reproduce a BOUND bound-state energy calculation (e.g. bound-basic) for an atom–rigid-rotor system and must specify the energy window (EMIN/EMAX), total angular momentum (JTOT), and symmetry/fixing options (IBFIX) in namelist input blocks.

## When NOT to use

- The goal is scattering observables (S matrices, cross sections, scattering lengths, resonances) rather than bound-state energies — that requires MOLSCAT, not BOUND namelist configuration
- The task is locating external-field values at which a bound state exists at a specified energy (e.g. Feshbach resonance positions) — that requires FIELD, a development of BOUND
- The coupled equations are not among the built-in basis types (e.g. atom + linear rigid rotor) and no plug-in basis-set suite is available

## Inputs

- BOUND namelist input file (e.g. bound-basic1.input) with &INPUT, &BASIS and &POTL blocks specifying interaction type, basis set, interaction potential, reduced mass, propagator, and energy range (EMIN, EMAX)
- Interaction potential data files (e.g. from the source_code/ and data/ directories of the molscat distribution)
- GNUmakefile adapted for the working environment (LIBS or LIBUTILS pointing at BLAS/LAPACK)

## Outputs

- Program output listing converged bound-state energies in cm-1 for each state number within [EMIN, EMAX] (e.g. bound-basic1.output: state 4 at -4.992074672 cm-1, state 5 at -1.413103892 cm-1)
- Compiled bound-basic executable built via make rules from GNUmakefile

## How to apply

Locate the example input file (e.g. examples/input/bound-basic1.input from the molscat/molscat distribution) and edit the namelist blocks read on unit 5, setting EMIN=-10.0 cm-1, EMAX=-1.0 cm-1, JTOT=1 and IBFIX=2 for the atom–rigid-rotor case in section 3.9 of the documentation. Before running, build the executable with GNU make, first adapting GNUmakefile: point the LIBS variable at optimised BLAS/LAPACK libraries, or download the Netlib routines and list the object files in LIBUTILS (GNUmakefile defaults to gfortran with optimisation level 0). Run the bound-basic executable with the input file piped to standard input (unit 5), and inspect the output for converged bound-state energies. Judge correctness by comparing reported energies against documented values — for the bound-basic1 case, state 4 at -4.992074672 cm-1 and state 5 at -1.413103892 cm-1.

## Related tools

- **bound 2025.0** (Executable that reads the namelist input on unit 5 and performs coupled-channel bound-state energy calculations for weakly bound atomic and molecular systems) — https://github.com/molscat/molscat
- **GNUmakefile / make** (Build system used to compile the bound-basic variant; LIBS/LIBUTILS variables must be adapted to available BLAS/LAPACK libraries before running) — https://github.com/molscat/molscat
- **LAPACK** (Linear algebra routines used by BOUND for the coupled-channel eigenvalue solves underlying the bound-state energies) — https://github.com/molscat/molscat
- **BLAS** (Basic linear algebra subroutines used by the LAPACK routines in BOUND; optimised versions are linked via the LIBS variable) — https://github.com/molscat/molscat
- **Fortran** (Language of the namelist input format and the BOUND source; the input file consists of Fortran namelist data blocks read on unit 5) — https://github.com/molscat/molscat

## Examples

```
make bound-basic; ./bound-basic < examples/input/bound-basic1.input > bound-basic1.output
```

## Evaluation signals

- Program output contains converged bound-state energies for the requested JTOT and IBFIX; for the bound-basic1 case (JTOT=1, IBFIX=2, EMIN=-10 to EMAX=-1 cm-1), state 4 must be -4.992074672 cm-1 and state 5 must be -1.413103892 cm-1, matching the section 3.9 documented values
- No bound states are reported outside the specified EMIN/EMAX window (-10.0 to -1.0 cm-1)
- The executable builds without missing-symbol errors from BLAS/LAPACK (dgemm, dsyevr, etc.), confirming LIBS/LIBUTILS were adapted correctly
- Output file saved in the same format as the examples/output/ reference files (e.g. bound-basic1.output) diff-comparable to the distributed example output

## Limitations

- GNUmakefile must be adapted for the local environment: if optimised BLAS/LAPACK are not in LIBS, the Netlib routines must be compiled into LIBUTILS, and the Compiler variable (default gfortran) may need changing
- Optimisation level is set to 0 with traceback on by default; only increase after verifying the code works
- Namelist input must match the Fortran namelist syntax exactly on unit 5; the built-in basis types cover atom + linear rigid rotor and a limited set of other systems, with other cases requiring a plug-in basis-set suite

## Evidence
