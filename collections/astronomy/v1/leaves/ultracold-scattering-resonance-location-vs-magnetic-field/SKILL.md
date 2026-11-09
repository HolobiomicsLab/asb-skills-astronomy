---
name: ultracold-scattering-resonance-location-vs-magnetic-field
description: Use when you need to map ultracold scattering observables (scattering
  length, effective range) against an external magnetic field to reveal resonance
  poles, e.g. an 800-900 G scan at 100 nK collision energy.
when_to_use_negative:
- The system involves reactive collisions — MOLSCAT is explicitly for non-reactive
  quantum scattering calculations only
- The goal is bound-state energies or locating Feshbach resonance positions directly
  — the BOUND or FIELD programs are the appropriate variants, not molscat scattering-length
  scans
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0176
tools:
- name: molscat 2025.0
  role: Coupled-channel non-reactive quantum scattering engine that computes low-energy
    s-wave scattering lengths as a function of an external magnetic field
  repo: https://github.com/molscat/molscat
- name: Fortran
  role: Source language of MOLSCAT 2025.0; the variant executable is compiled from
    Fortran 90 source with gfortran or ifort
  repo: https://github.com/molscat/molscat
- name: LAPACK
  role: Linear algebra routines (dgesv, dsyevr, dsytrf, dsytri, dsyr2k, dlamch, ilaenv,
    lsame) required to build the molscat variant
- name: BLAS
  role: Basic linear algebra subroutines (daxpy, dcopy, dgemm, dgemv, dgesv, dscal,
    dswap, dsymm, idamax) linked via LIBS in GNUmakefile
provenance:
  source_task_ids:
  - task_005
  source_content_hash: sha256:b5c73790705db6966f939ff7d53643bcf9189a22dbf88f7d4cedf6d336be4478
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0176
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# ultracold-scattering-resonance-location-vs-magnetic-field

## Summary

Reproduce field-dependent low-energy scattering calculations with a task-specific MOLSCAT 2025.0 variant (e.g. molscat-Rb2) to extract s-wave scattering lengths as a function of magnetic field and locate wide poles and narrow Feshbach resonances. Grounded in MOLSCAT's coupled-channel capability to 'locate and characterize scattering resonances as a function of an external variable such as the magnetic field'.

## When to use

Use when you need to map ultracold scattering observables (scattering length, effective range) against an external magnetic field to reveal resonance poles, e.g. an 800-900 G scan at 100 nK collision energy.

## When NOT to use

- The system involves reactive collisions — MOLSCAT is explicitly for non-reactive quantum scattering calculations only
- The goal is bound-state energies or locating Feshbach resonance positions directly — the BOUND or FIELD programs are the appropriate variants, not molscat scattering-length scans

## Inputs

- Fortran 90 source code of MOLSCAT 2025.0 (source_code directory with GNUmakefile)
- MOLSCAT namelist input file (e.g. molscat-basic_Rb2.input) with &INPUT, &BASIS and &POTL blocks defining the alkali-metal + alkali-metal basis with hyperfine structure in a magnetic field

## Outputs

- MOLSCAT output containing per-field-point s-wave scattering lengths and effective ranges for low-energy scattering
- CSV table of magnetic field (G) vs scattering length (e.g. rb2_scattering_length_vs_field.csv)
- Plot of the real part of the scattering length vs magnetic field showing resonance poles

## How to apply

Build the variant executable (e.g. molscat-Rb2) from the Fortran 90 source with GNUmakefile, placing optimised BLAS/LAPACK routines (daxpy, dcopy, dgemm, dgemv, dgesv, dscal, dswap, dsymm, idamax; dgesv, dlamch, dsyevr, dsyr2k, dsytrf, dsytri, ilaenv, lsame) in LIBS. Run it on a namelist input file (&INPUT, &BASIS, &POTL) that sets up the alkali-metal atom + alkali-metal atom basis with hyperfine structure in an external magnetic field, scanning 800 to 900 G in 1 G steps at 100 nK collision energy. For each of the 101 field points, extract the s-wave scattering length (and effective range) MOLSCAT reports for low-energy scattering. Assemble a table of field (G) vs scattering length (e.g. rb2_scattering_length_vs_field.csv) and inspect the curve for a wide pole near 850 G and a narrow resonance near 820 G.

## Related tools

- **molscat 2025.0** (Coupled-channel non-reactive quantum scattering engine that computes low-energy s-wave scattering lengths as a function of an external magnetic field) — https://github.com/molscat/molscat
- **Fortran** (Source language of MOLSCAT 2025.0; the variant executable is compiled from Fortran 90 source with gfortran or ifort) — https://github.com/molscat/molscat
- **LAPACK** (Linear algebra routines (dgesv, dsyevr, dsytrf, dsytri, dsyr2k, dlamch, ilaenv, lsame) required to build the molscat variant)
- **BLAS** (Basic linear algebra subroutines (daxpy, dcopy, dgemm, dgemv, dgesv, dscal, dswap, dsymm, idamax) linked via LIBS in GNUmakefile)

## Examples

```
make molscat-Rb2 && ./molscat-Rb2 < molscat-basic_Rb2.input
```

## Evaluation signals

- The run completes without error for all 101 field points (800-900 G in 1 G steps)
- The computed scattering-length curve contains a pole within a few gauss of 850 G, matching section 3.10.1 behaviour
- A narrow resonance feature is visible within a few gauss of 820 G in the scattering-length plot
- The CSV table has one scattering-length value per scanned field point with no missing entries

## Limitations

- Optimisation level is set to 0 with traceback switched on by default in GNUmakefile; increase optimisation only after verifying the code works
- If optimised BLAS/LAPACK are unavailable, the required routines must be downloaded from Netlib and placed in LIBUTILS
- MOLSCAT handles only non-reactive collisions; simple atom-molecule and molecule-molecule types are coded internally, other collision types need plug-in routines

## Evidence
