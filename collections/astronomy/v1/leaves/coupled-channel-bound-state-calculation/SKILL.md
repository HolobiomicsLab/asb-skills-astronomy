---
name: coupled-channel-bound-state-calculation
description: Use when the goal is to locate bound-state eigenvalues of an atom–molecule
  or molecule–molecule system (or field-dependent threshold crossings) from namelist-defined
  &INPUT/&BASIS/&POTL coupled-channel inputs.
when_to_use_negative:
- The task is scattering/cross-section calculation (S matrices, integral cross sections,
  resonance eigenphase scans) rather than bound states — use MOLSCAT instead
- The system requires reactive scattering or dynamics beyond the built-in plug-in
  basis suites (atom + linear rotor, vibrating diatom, rotor+rotor, tops, corrugated
  surface, magnetic-field alkali suites)
- Only qualitative/analytic bound-state estimates are needed and no compiled BOUND
  executable or BLAS/LAPACK toolchain can be built
edam_topics:
- http://edamontology.org/topic_3332
- http://edamontology.org/topic_3318
tools:
- name: bound
  role: Core executable that solves the coupled equations with bound-state boundary
    conditions to converge bound-state eigenvalues
  repo: https://github.com/molscat/molscat
- name: field
  role: BOUND-derived variant that scans an external variable (e.g. magnetic field
    in Gauss) to find where a bound state exists at a specified energy
  repo: https://github.com/molscat/molscat
- name: LAPACK
  role: Linear algebra routines (dsyevr, dsytrf, dsytri, dgesv) used by the programs
    wherever possible
  repo: https://github.com/molscat/molscat
- name: BLAS
  role: Basic linear algebra subroutines (dgemm, dsymm, daxpy, etc.) underpinning
    the LAPACK calls; must be supplied via LIBS or LIBUTILS
  repo: https://github.com/molscat/molscat
- name: gfortran
  role: Default compiler set in the GNUmakefile Compiler variable for building bound-
    variants
  repo: https://github.com/molscat/molscat
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_006
  - task_007
  - task_003
  - task_001
  source_content_hash: sha256:7ef934c2ae61f51a53fdf5b0f21939b77e42e5187d45a4478c966036ebfb68db
  source_papers:
  - doi: 10.48550/arxiv.1903.06755
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/molscat/molscat
  tools:
  - BLAS
  - BOUND
  - GNUmakefile
  - GNUmakefile (GNU make)
  - LAPACK
  - bound
  - field
  - gfortran
  - molscat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3332
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# coupled-channel-bound-state-calculation

## Summary

Compute converged bound-state energies of weakly bound atomic and molecular complexes by solving the coupled-channel radial equations with the BOUND/FIELD suite (version 2025.0), including bound states at specified energies relative to threshold as a function of an external field (e.g. magnetic Feshbach resonance positions).

## When to use

Use when the goal is to locate bound-state eigenvalues of an atom–molecule or molecule–molecule system (or field-dependent threshold crossings) from namelist-defined &INPUT/&BASIS/&POTL coupled-channel inputs.

## When NOT to use

- The task is scattering/cross-section calculation (S matrices, integral cross sections, resonance eigenphase scans) rather than bound states — use MOLSCAT instead
- The system requires reactive scattering or dynamics beyond the built-in plug-in basis suites (atom + linear rotor, vibrating diatom, rotor+rotor, tops, corrugated surface, magnetic-field alkali suites)
- Only qualitative/analytic bound-state estimates are needed and no compiled BOUND executable or BLAS/LAPACK toolchain can be built

## Inputs

- Namelist input file (bound-*.input) on unit 5 with &INPUT, &BASIS and &POTL blocks (e.g. EMIN=-10.0, EMAX=-1.0 cm-1, JTOT=1, IBFIX=2)
- Interaction potential definition or plug-in potential data files (e.g. Mg + NH RKHS-interpolated Soldan et al. potential, H2 + H2 data files in data/)
- GNUmakefile adapted with LIBS (optimised BLAS/LAPACK) or LIBUTILS (Netlib object files) and Compiler setting

## Outputs

- Executable output listing converged bound-state energies with state numbers (e.g. 'CONVERGED ON STATE NUMBER 4' at -4.992074672 cm-1)
- For FIELD runs: external-field values (Gauss) at which a bound level crosses a specified threshold energy (e.g. 851.8761789 G for 85Rb2, ~387.28 G for Mg + NH)
- Output file saved as bound-*.output / field-*.output in examples/output/ style, plus a verification record (e.g. bound-2chan-LJ-verification.txt)

## How to apply

Adapt GNUmakefile to the local environment, setting LIBS to optimised BLAS/LAPACK (or LIBUTILS to Netlib object files) and the Compiler variable (default gfortran), then build a variant executable with a make rule such as 'make bound-basic' following the prefix convention (bound-, molscat-, field-). Supply the namelist input on unit 5 (standard input) via &INPUT, &BASIS and &POTL blocks specifying interaction type, basis, potential, reduced mass, propagator (e.g. LDMD/AIRY log-derivative) and the energy or field search window (e.g. EMIN=-10 to EMAX=-1 cm-1, JTOT=1, IBFIX=2). Run the executable and inspect the output for 'CONVERGED ON STATE NUMBER n' lines reporting the bound-state energies (e.g. -4.992074672 cm-1 for bound-basic1) or, for FIELD, the field values where a bound level crosses the specified threshold energy. Validate by diffing the produced output against the reference file in examples/output/ and against values documented in the manual (e.g. sections 3.9, 3.10.2, 14.3.2), and save the log in the style of examples/output/ files.

## Related tools

- **bound** (Core executable that solves the coupled equations with bound-state boundary conditions to converge bound-state eigenvalues) — https://github.com/molscat/molscat
- **field** (BOUND-derived variant that scans an external variable (e.g. magnetic field in Gauss) to find where a bound state exists at a specified energy) — https://github.com/molscat/molscat
- **LAPACK** (Linear algebra routines (dsyevr, dsytrf, dsytri, dgesv) used by the programs wherever possible) — https://github.com/molscat/molscat
- **BLAS** (Basic linear algebra subroutines (dgemm, dsymm, daxpy, etc.) underpinning the LAPACK calls; must be supplied via LIBS or LIBUTILS) — https://github.com/molscat/molscat
- **gfortran** (Default compiler set in the GNUmakefile Compiler variable for building bound- variants) — https://github.com/molscat/molscat

## Examples

```
make bound-basic && ./bound-basic < examples/input/bound-basic1.input > examples/output/bound-basic1.output
```

## Evaluation signals

- Output contains 'CONVERGED ON STATE NUMBER n' lines for each located bound level within the specified EMIN/EMAX window
- Converged energies match the reference output in examples/output/ (e.g. bound-2chan-LJ: approximately -58.2 and -12.1 cm-1) within numerical tolerance
- Match to documented manual values (e.g. -4.992074672 and -1.413103892 cm-1 for bound-basic1; 851.8761789, 819.5625939, 810.9471815 G for field-basic_Rb2)
- Build succeeded with LIBS pointing to optimised BLAS/LAPACK (or LIBUTILS Netlib objects) and no missing-routine link errors
- Verification record (e.g. bound-2chan-LJ-verification.txt) written and run log saved in examples/output/ style

## Limitations

- GNUmakefile must be adapted per environment (LIBS vs LIBUTILS); if date/time intrinsics date_and_time or cpu_time are unavailable, GDATE/GTIME/GCLOCK must be replaced locally or stubbed
- Optimisation is set to 0 with traceback on by default in COMPILE.f; only increase after the code is verified to work
- If EXECDIR is set to the current directory, the commented circular-dependency line in GNUmakefile must be uncommented to avoid make warnings; object/executable directories named in OBJDIR/EXECDIR must exist
- Version 2025.0 removed the fixed storage array X (common block MEMORY) in favour of allocatable arrays — older workflows assuming X/MX may not apply

## Evidence

- [abstract] BOUND is a general-purpose package for performing calculations of bound-state energies in weakly bound atomic and molecular systems using coupled-channel methods.: "BOUND is a general-purpose package for performing calculations of bound-state energies in weakly bound atomic and molecular systems using coupled-channel methods."
- [abstract] input on unit 5 namelists: "The main input file for any of the programs is read on unit 5 (standard input), and consists of several blocks of namelist data"
- [intro] FIELD purpose: [evidence span withheld — anchor status: not_found]
- [readme] makefile adaptation and BLAS/LAPACK: "GNUmakefile first needs to be adapted for your own working environment.  MOLSCAT, BOUND and FIELD executables make use of BLAS and LAPACK library routines."
- [methods] make rule convention: "They may be made with Linux commands such as
make molscat-H2_H2"
