---
name: triplet-sigma-diatom-magnetic-field-basis-setup
description: Use when calculating bound states or scattering of a closed-shell atom
  with a triplet-Sigma diatom (e.g. Mg + NH) in a magnetic field, or when locating
  field values where a bound level crosses a specified energy near threshold.
when_to_use_negative:
- The system is not a closed-shell atom + triplet-Sigma diatom (e.g. it is an alkali-metal
  atom pair, which uses the dedicated alkali-metal + alkali-metal hyperfine basis
  suite instead)
- The goal is ordinary bound-state energies at a fixed field or scattering cross sections
  rather than locating field values at specified binding energies (use BOUND or MOLSCAT,
  not FIELD)
- No external field is involved and no plug-in basis suite is needed, so a standard
  internal atom + diatom basis would suffice
edam_operation: http://edamontology.org/operation_2479
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: field 2025.0
  role: Executable that locates external magnetic fields at which a bound state exists
    with a specified energy, using the triplet-Sigma diatom plug-in basis suite
  repo: https://github.com/molscat/molscat
- name: Fortran
  role: Language of the plug-in basis suite and FIELD source, compiled with gfortran
    via GNUmakefile
  repo: https://github.com/molscat/molscat
- name: LAPACK
  role: Linear algebra routines (e.g. dsyevr, dsyr2k, dsytrf) required by the coupled-channel
    solver, linked through LIBS or LIBUTILS
- name: BLAS
  role: Basic linear algebra subroutines (e.g. dgemm, dsymm) used by the LAPACK routines
    in the propagation
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:5897ffcbeb938105d3749897f9d7ebc67ece9734972a6536593a01555a23bc00
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
  - field 2025.0
  edam_operation: http://edamontology.org/operation_2479
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# triplet-sigma-diatom-magnetic-field-basis-setup

## Summary

Sets up a closed-shell atom + triplet-Sigma diatom plug-in basis suite under an external magnetic field for FIELD/BOUND/molscat executables, using the RKHS-interpolated Soldan et al. Mg + NH potential to locate magnetically tunable bound states (e.g. Feshbach resonance positions) as a function of field strength.

## When to use

Use when calculating bound states or scattering of a closed-shell atom with a triplet-Sigma diatom (e.g. Mg + NH) in a magnetic field, or when locating field values where a bound level crosses a specified energy near threshold.

## When NOT to use

- The system is not a closed-shell atom + triplet-Sigma diatom (e.g. it is an alkali-metal atom pair, which uses the dedicated alkali-metal + alkali-metal hyperfine basis suite instead)
- The goal is ordinary bound-state energies at a fixed field or scattering cross sections rather than locating field values at specified binding energies (use BOUND or MOLSCAT, not FIELD)
- No external field is involved and no plug-in basis suite is needed, so a standard internal atom + diatom basis would suffice

## Inputs

- field-Mg_NH.input namelist input file (from examples/input/) with &INPUT, &BASIS and &POTL blocks
- Mg + NH potential data files from the data/ directory (RKHS-interpolated Soldan et al. potential)
- Fortran source code files (.f/.f90) in source_code/ and GNUmakefile
- Target binding energies of the bound level relative to threshold (0, 20 and 40 MHz) and field search range (0 to 400 G)

## Outputs

- field-Mg_NH executable built via make
- field-Mg_NH.output in examples/output/ reporting the magnetic field values at which the bound state exists at each specified energy
- Located threshold-crossing field (expected near 387.28 G)

## How to apply

Adapt GNUmakefile for the local environment (set LIBS to optimised BLAS/LAPACK or LIBUTILS to Netlib object files; the default compiler is gfortran) and build the field-Mg_NH executable with make. Supply the example input file field-Mg_NH.input from examples/input/, which configures the closed-shell atom + triplet-Sigma diatom plug-in basis suite in a magnetic field via the &INPUT, &BASIS and &POTL namelist blocks and selects the RKHS-interpolated Soldan et al. Mg + NH potential from the data directory. Run FIELD with LDMD/AIRY log-derivative propagation and TOLHIL=0 (full convergence) to locate the magnetic field values at which the bound level lies 0, 20 and 40 MHz below the lowest scattering threshold. Inspect field-Mg_NH.output in examples/output/ and verify the located crossing fields reproduce the reference threshold crossing near 387.28 G. The rationale is that FIELD, a development of BOUND, inverts the bound-state problem to find field values at specified binding energies, which is how magnetically tunable Feshbach resonance positions are located.

## Related tools

- **field 2025.0** (Executable that locates external magnetic fields at which a bound state exists with a specified energy, using the triplet-Sigma diatom plug-in basis suite) — https://github.com/molscat/molscat
- **Fortran** (Language of the plug-in basis suite and FIELD source, compiled with gfortran via GNUmakefile) — https://github.com/molscat/molscat
- **LAPACK** (Linear algebra routines (e.g. dsyevr, dsyr2k, dsytrf) required by the coupled-channel solver, linked through LIBS or LIBUTILS)
- **BLAS** (Basic linear algebra subroutines (e.g. dgemm, dsymm) used by the LAPACK routines in the propagation)

## Examples

```
make field-Mg_NH && ./field-Mg_NH < examples/input/field-Mg_NH.input > examples/output/field-Mg_NH.output
```

## Evaluation signals

- field-Mg_NH.output is produced in examples/output/ after the run
- The three located crossing fields for 0, 20 and 40 MHz below threshold are mutually consistent with the bound level trajectory
- The located threshold crossing reproduces the reference value near 387.28 G (as in manual sections 14.2.9 and 14.3.1) within the convergence implied by TOLHIL=0
- The executable builds cleanly with make after LIBS/LIBUTILS and Compiler (gfortran) are set in GNUmakefile
- The input file correctly resolves the Mg + NH potential from the data/ directory without potential-construction errors

## Limitations

- GNUmakefile must be adapted to the local environment first: without optimised BLAS/LAPACK in LIBS, Netlib object files must be supplied in LIBUTILS or the build fails
- Optimisation level is set at 0 with traceback on by default; increasing optimisation is only advised after confirming the code works
- The result is specific to the Soldan et al. Mg + NH potential and the closed-shell atom + triplet-Sigma diatom basis suite; other systems need different plug-in suites or data files

## Evidence
