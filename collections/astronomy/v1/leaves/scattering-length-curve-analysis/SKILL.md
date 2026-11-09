---
name: scattering-length-curve-analysis
description: Use when a low-energy (e.g. 100 nK) atom–atom collision calculation reports
  scattering lengths as a function of a scanned external variable such as a magnetic
  field, and you must locate poles/resonances in the curve.
when_to_use_negative:
- The collision system is reactive or involves potentials outside the plug-in basis
  suites (e.g. closed-shell atom + triplet-Sigma diatom without a field) — MOLSCAT
  is for non-reactive scattering
- The goal is to locate resonance positions directly rather than map the curve — FIELD
  locates fields at which a bound state exists at a specified energy and may be more
  appropriate
- The collision energy is not in the low-energy/ultracold regime where a scattering
  length is the meaningful observable
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: molscat 2025.0
  role: Performs the coupled-channel non-reactive quantum scattering calculations
    and reports s-wave scattering lengths and effective ranges for low-energy scattering
  repo: https://github.com/molscat/molscat
- name: molscat-Rb2 (MOLSCAT variant)
  role: Special-purpose executable built with the alkali-metal atom + alkali-metal
    atom basis-set suite including hyperfine structure in a magnetic field
  repo: https://github.com/molscat/molscat
- name: Fortran
  role: Language of the source code compiled via GNUmakefile to produce the executables
  repo: https://github.com/molscat/molscat
- name: LAPACK
  role: Linear algebra routines linked via LIBS and used by the programs wherever
    possible
  repo: https://github.com/molscat/molscat
- name: BLAS
  role: Basic linear algebra subroutines underlying the LAPACK routines; optimised
    versions should be included in LIBS
  repo: https://github.com/molscat/molscat
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:a9177259cb9db4d81c1cf9c74817ac72297845f923afa67f19b109d6db297ce8
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
  - molscat-Rb2 (MOLSCAT variant)
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# scattering-length-curve-analysis

## Summary

Compute and inspect the field-dependent s-wave scattering length from MOLSCAT 2025.0 low-energy scattering output to locate Feshbach resonances (poles) and narrow resonances as a function of an external magnetic field.

## When to use

Use when a low-energy (e.g. 100 nK) atom–atom collision calculation reports scattering lengths as a function of a scanned external variable such as a magnetic field, and you must locate poles/resonances in the curve.

## When NOT to use

- The collision system is reactive or involves potentials outside the plug-in basis suites (e.g. closed-shell atom + triplet-Sigma diatom without a field) — MOLSCAT is for non-reactive scattering
- The goal is to locate resonance positions directly rather than map the curve — FIELD locates fields at which a bound state exists at a specified energy and may be more appropriate
- The collision energy is not in the low-energy/ultracold regime where a scattering length is the meaningful observable

## Inputs

- Namelist input file (e.g. molscat-basic_Rb2.input) with &INPUT, &BASIS and &POTL blocks defining the alkali-metal atom pair, hyperfine structure, magnetic-field scan (800–900 G, 1 G steps) and 100 nK collision energy
- Built MOLSCAT 2025.0 executable variant with field-dependent basis suite (molscat-Rb2)
- Optimised BLAS/LAPACK libraries in the GNUmakefile LIBS variable

## Outputs

- MOLSCAT output listing per-field low-energy s-wave scattering length (and effective range)
- Table of field (G) vs scattering length saved as CSV (e.g. rb2_scattering_length_vs_field.csv)
- Plot of real part of scattering length vs magnetic field showing wide pole (~850 G) and narrow resonance (~820 G)

## How to apply

Build the MOLSCAT variant (e.g. molscat-Rb2) with GNUmakefile, linking optimised BLAS (daxpy, dcopy, dgemm, dgemv, dgesv, dscal, dswap, dsymm, idamax) and LAPACK (dgesv, dlamch, dsyevr, dsyr2k, dsytrf, dsytri, ilaenv, lsame) routines in LIBS. Run the variant on a namelist input file (&INPUT, &BASIS, &POTL) that sets up the alkali-metal atom + alkali-metal atom basis with hyperfine structure in an external magnetic field, scanning the field (e.g. 800–900 G in 1 G steps) at a fixed collision energy (e.g. 100 nK). For each field point, extract the s-wave scattering length (and effective range) that MOLSCAT reports for low-energy scattering. Assemble field (G) vs scattering length into a CSV (e.g. rb2_scattering_length_vs_field.csv) and inspect the real part of the scattering length versus field: a wide resonance appears as a pole, a narrow resonance as a small feature. Decision point: a pole is confirmed when the scattering length diverges within a few gauss of the expected field value.

## Related tools

- **molscat 2025.0** (Performs the coupled-channel non-reactive quantum scattering calculations and reports s-wave scattering lengths and effective ranges for low-energy scattering) — https://github.com/molscat/molscat
- **molscat-Rb2 (MOLSCAT variant)** (Special-purpose executable built with the alkali-metal atom + alkali-metal atom basis-set suite including hyperfine structure in a magnetic field) — https://github.com/molscat/molscat
- **Fortran** (Language of the source code compiled via GNUmakefile to produce the executables) — https://github.com/molscat/molscat
- **LAPACK** (Linear algebra routines linked via LIBS and used by the programs wherever possible) — https://github.com/molscat/molscat
- **BLAS** (Basic linear algebra subroutines underlying the LAPACK routines; optimised versions should be included in LIBS) — https://github.com/molscat/molscat

## Examples

```
make molscat-Rb2 && ./molscat-Rb2 < molscat-basic_Rb2.input
```

## Evaluation signals

- The run completes without error for all 101 field points in the 800–900 G scan
- The scattering-length curve contains a pole positioned within a few gauss of 850 G
- A narrow resonance feature is visible within a few gauss of 820 G
- The CSV contains one scattering-length value per field value with no missing field steps
- Behaviour matches that reported in section 3.10.1 of the MOLSCAT/BOUND/FIELD documentation

## Limitations

- Optimisation level in GNUmakefile is set at 0 with traceback switched on; increase optimisation only after verifying the code works
- If optimised BLAS/LAPACK are unavailable, the requisite routines must be downloaded from Netlib and included as object files in LIBUTILS
- The executables rely on f90 intrinsics date_and_time and cpu_time (GDATE, GTIME, GCLOCK); if unavailable they must be replaced with local routines
- GNUmakefile places object/executable files in OBJDIR/EXECDIR (default current directory); changing EXECDIR requires uncommenting the PROGS rule to avoid circular-dependency issues

## Evidence
