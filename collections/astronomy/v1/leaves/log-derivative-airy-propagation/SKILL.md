---
name: log-derivative-airy-propagation
description: Use when solving coupled-channel scattering or bound-state equations
  for ultracold atom–molecule systems, or when locating external field values where
  a bound level crosses a specified energy (e.g. Feshbach resonance positions).
when_to_use_negative:
- The goal is reactive (rearrangement) scattering — the suite is explicitly for non-reactive
  quantum scattering calculations
- Only bound-state energies at fixed geometry/field are needed and no field search
  is required — use BOUND directly instead of the FIELD development of BOUND
- An approximate treatment is acceptable and the close-coupling cost is prohibitive
  — although coupled-states/helicity-decoupling approximations exist, the TOLHIL=0
  full-convergence setting of this skill assumes exact close-coupling
tools:
- name: FIELD 2025.0
  role: Executable that locates external field values at which a bound state exists
    with a specified energy, using log-derivative propagation with bound-state boundary
    conditions
  repo: https://github.com/molscat/molscat
- name: BOUND 2025.0
  role: Base package whose bound-state boundary-condition machinery FIELD develops;
    solves the same coupled equations with the same plug-in routines
  repo: https://github.com/molscat/molscat
- name: Fortran
  role: Language of the executables; gfortran is set in the Compiler variable of GNUmakefile
  repo: https://github.com/molscat/molscat
- name: LAPACK
  role: Linear algebra routines used wherever possible by the propagation and eigenvalue
    steps; must be linked via LIBS or supplied from Netlib via LIBUTILS
  repo: https://github.com/molscat/molscat
- name: BLAS
  role: Basic linear algebra subroutines underlying LAPACK (daxpy, dgemm, dsymm, etc.)
    required at link time for the propagator executables
  repo: https://github.com/molscat/molscat
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:2b6265775e4602137f8a4ced0c2f4bef7f74d518aebfa171f07eb7ffdd0007f6
  source_papers:
  - doi: 10.48550/arxiv.1903.06755
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/molscat/molscat
  tools:
  - BLAS
  - BOUND 2025.0
  - FIELD 2025.0
  - Fortran
  - LAPACK
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# log-derivative-airy-propagation

## Summary

Use the LDMD/AIRY log-derivative propagator implemented in the MOLSCAT/BOUND/FIELD suite to propagate coupled-channel equations and, in FIELD, to locate magnetic field values at which a bound state exists at a specified energy (e.g. the Mg + NH threshold crossing near 387.28 G).

## When to use

Use when solving coupled-channel scattering or bound-state equations for ultracold atom–molecule systems, or when locating external field values where a bound level crosses a specified energy (e.g. Feshbach resonance positions).

## When NOT to use

- The goal is reactive (rearrangement) scattering — the suite is explicitly for non-reactive quantum scattering calculations
- Only bound-state energies at fixed geometry/field are needed and no field search is required — use BOUND directly instead of the FIELD development of BOUND
- An approximate treatment is acceptable and the close-coupling cost is prohibitive — although coupled-states/helicity-decoupling approximations exist, the TOLHIL=0 full-convergence setting of this skill assumes exact close-coupling

## Inputs

- Namelist input file (e.g. field-Mg_NH.input) with &INPUT, &BASIS and &POTL blocks specifying basis suite, potential, propagator (LDMD/AIRY), TOLHIL, and target energies/fields
- Potential data files (e.g. RKHS-interpolated Soldan et al. Mg + NH potential from the data directory)
- Field range and target bound-state energies below the scattering threshold (e.g. 0–400 G; 0, 20, 40 MHz below threshold)

## Outputs

- Program output file (e.g. field-Mg_NH.output in examples/output/) listing the magnetic field values at which the bound level has each specified energy
- Converged bound-state/field crossing values for comparison against reference values (e.g. crossing near 387.28 G)

## How to apply

Build the relevant executable (e.g. field-Mg_NH) after adapting the GNUmakefile variable LIBS with optimised BLAS/LAPACK libraries, or LIBUTILS with Netlib object files, keeping the default gfortran compiler. Supply a namelist input file (&INPUT, &BASIS, &POTL) that sets the interaction type, the plug-in basis suite (e.g. closed-shell atom + triplet-Sigma diatom in a magnetic field), the RKHS-interpolated Soldan et al. potential, the reduced mass, and the target energies. Select the LDMD/AIRY log-derivative propagation method and set TOLHIL=0 for full convergence, then let FIELD loop over magnetic field strengths to find where the bound level lies at the specified energies (0, 20 and 40 MHz below threshold for the Mg + NH example). The rationale is that the log-derivative formulation with Airy-function propagation is numerically stable for the wide range of channel energies encountered in close-coupled ultracold calculations, and FIELD solves for the field at which bound-state boundary conditions are met. Inspect the .output file and compare the located crossing fields against the reference value (threshold crossing near 387.28 G) within the convergence implied by TOLHIL=0.

## Related tools

- **FIELD 2025.0** (Executable that locates external field values at which a bound state exists with a specified energy, using log-derivative propagation with bound-state boundary conditions) — https://github.com/molscat/molscat
- **BOUND 2025.0** (Base package whose bound-state boundary-condition machinery FIELD develops; solves the same coupled equations with the same plug-in routines) — https://github.com/molscat/molscat
- **Fortran** (Language of the executables; gfortran is set in the Compiler variable of GNUmakefile) — https://github.com/molscat/molscat
- **LAPACK** (Linear algebra routines used wherever possible by the propagation and eigenvalue steps; must be linked via LIBS or supplied from Netlib via LIBUTILS) — https://github.com/molscat/molscat
- **BLAS** (Basic linear algebra subroutines underlying LAPACK (daxpy, dgemm, dsymm, etc.) required at link time for the propagator executables) — https://github.com/molscat/molscat

## Examples

```
make field-Mg_NH; ./field-Mg_NH < examples/input/field-Mg_NH.input > examples/output/field-Mg_NH.output
```

## Evaluation signals

- field-Mg_NH.output is produced in examples/output/ after the run
- The three located crossing fields reproduce the reference values for the bound level at 0, 20 and 40 MHz below threshold
- The located threshold crossing agrees with the reference value near 387.28 G reported in manual sections 14.2.9 and 14.3.1
- TOLHIL=0 was set so the result is fully converged, i.e. tighter TOLHIL settings do not shift the crossing fields
- The executable built successfully with make using the adapted GNUmakefile (LIBS/LIBUTILS, compiler setting)

## Limitations

- GNUmakefile must be adapted to the local environment (LIBS/LIBUTILS, Compiler) before building; optimisation level defaults to 0 with traceback on for safety and can only be raised after validating the code
- If GDATE/GTIME/GCLOCK intrinsic date-and-time routines are unavailable they must be replaced with local routines or stubs
- Exifile/EXECDIR/OBJDIR directories must exist if changed; a commented line in GNUmakefile must be uncommented when EXECDIR is not the current directory
- Close-coupling with no dynamical approximations is the accurate mode; approximate methods (coupled states, helicity decoupling) trade accuracy for speed and would change the convergence guarantees

## Evidence
