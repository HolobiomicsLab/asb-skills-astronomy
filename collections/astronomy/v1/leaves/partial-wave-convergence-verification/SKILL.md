---
name: partial-wave-convergence-verification
description: Use when a molscat coupled-channel run must be confirmed converged over
  the JTOT range (e.g. JTOT 10 to 20 step 10) and state-to-state cross sections SIG(j,j')
  must match reference values at a fixed collision energy such as 1250 cm-1.
when_to_use_negative:
- The goal is bound-state energies (use BOUND) or locating external-field values for
  bound states (use FIELD), not scattering cross sections
- Differential cross sections, transport/relaxation/Senftleben-Beenakker cross sections,
  or resonance fitting are needed — these require post-processors (DCS, SBE, RESFIT)
  reading S-matrix files, not the basic cross-section check
- A reactive collision or collision type not coded internally and not covered by an
  available plug-in routine is involved
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: molscat
  role: Runs the coupled-channel scattering calculation, looping over total angular
    momentum (partial wave) to produce elastic and inelastic integral cross sections
  repo: https://github.com/molscat/molscat
- name: Fortran
  role: Source language compiled (gfortran) to build the molscat-basic executable
- name: LAPACK
  role: Linear algebra routines linked into the executable for solving the coupled-channel
    equations
- name: BLAS
  role: Basic linear algebra subroutines used by LAPACK routines; optimised versions
    go in LIBS during the build
provenance:
  source_task_ids:
  - task_001
  source_content_hash: sha256:fc93ab0c82efbf4cbd28a06c4e5ffbc5f7eb13bbf453919268946dc491d1d130
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# partial-wave-convergence-verification

## Summary

Verifies partial-wave (total angular momentum JTOT) convergence of molscat scattering calculations by looping over total angular momentum to compute elastic and inelastic integral cross sections, and comparing extracted SIG(j,j') values against documented reference outputs (e.g. molscat-basic1, section 3.8.1).

## When to use

Use when a molscat coupled-channel run must be confirmed converged over the JTOT range (e.g. JTOT 10 to 20 step 10) and state-to-state cross sections SIG(j,j') must match reference values at a fixed collision energy such as 1250 cm-1.

## When NOT to use

- The goal is bound-state energies (use BOUND) or locating external-field values for bound states (use FIELD), not scattering cross sections
- Differential cross sections, transport/relaxation/Senftleben-Beenakker cross sections, or resonance fitting are needed — these require post-processors (DCS, SBE, RESFIT) reading S-matrix files, not the basic cross-section check
- A reactive collision or collision type not coded internally and not covered by an available plug-in routine is involved

## Inputs

- molscat-basic executable built from source_code with GNUmakefile (gfortran + BLAS/LAPACK in LIBS)
- Example input file examples/input/molscat-basic1.input (ITYP=1, URED=20, LJ 12-6 + R^-6 potential, energy 1250 cm-1, JTOT 10 to 20 step 10)

## Outputs

- molscat-basic1.output log containing state-to-state integral cross sections SIG(j,j') (e.g. SIG(1,1), SIG(2,1))
- Total inelastic integral cross sections per level (e.g. 3.72239E-02, 8.77864E-03, 7.47803E-05 A^2)

## How to apply

Build the molscat-basic executable via GNUmakefile with gfortran, placing optimised BLAS and LAPACK in LIBS (or Netlib object files in LIBUTILS). Run molscat-basic with the example input (ITYP=1 atom + linear molecule, URED=20, Lennard-Jones 12-6 plus R^-6 potential, single energy 1250 cm-1, JTOT = 10 to 20 step 10), writing the log to molscat-basic1.output. Extract partial-wave-converged state-to-state integral cross sections SIG(j,j') from the output, including elastic SIG(1,1) and inelastic SIG(2,1). MOLSCAT loops over total angular momentum (partial wave) internally to accumulate elastic and inelastic integral cross sections, so extending or checking the JTOT range tests whether higher partial waves still contribute. The task succeeds when every cross section matches the documented reference (section 3.8.1: SIG(1,1)=1.81057 A^2, SIG(2,1)=3.722327E-02 A^2, total inelastic 3.72239E-02 A^2) to the printed precision.

## Related tools

- **molscat** (Runs the coupled-channel scattering calculation, looping over total angular momentum (partial wave) to produce elastic and inelastic integral cross sections) — https://github.com/molscat/molscat
- **Fortran** (Source language compiled (gfortran) to build the molscat-basic executable)
- **LAPACK** (Linear algebra routines linked into the executable for solving the coupled-channel equations)
- **BLAS** (Basic linear algebra subroutines used by LAPACK routines; optimised versions go in LIBS during the build)

## Examples

```
make molscat-basic && ./molscat-basic < examples/input/molscat-basic1.input > molscat-basic1.output
```

## Evaluation signals

- SIG(1,1) elastic cross section equals 1.81057 A^2 at 1250 cm-1
- SIG(2,1) inelastic cross section equals 3.722327E-02 A^2 at 1250 cm-1
- Total inelastic integral cross sections from level 1 match 3.72239E-02, 8.77864E-03 and 7.47803E-05 A^2 (section 3.8.1 reference values)
- Every cross section in molscat-basic1.output matches the corresponding documented reference value to the printed precision
- The JTOT loop (10 to 20 step 10) completed with no convergence warnings in the log

## Limitations

- Reference values are specific to the model atom + homonuclear rigid rotor system with the LJ 12-6 + R^-6 potential, URED=20 and energy 1250 cm-1; they do not transfer to other systems
- The build requires adapting GNUmakefile to the environment (compiler and BLAS/LAPACK availability); missing optimised libraries require Netlib object files in LIBUTILS
- Optimisation level is set to 0 with traceback on by default; results should be checked before raising optimisation
- Cross sections are only as converged as the JTOT range specified in the input (here 10 to 20 step 10)

## Evidence

- [other] reference output reports total inelastic integral cross section of 3.72239E-02 A^2: [evidence span withheld — anchor status: not_found]
