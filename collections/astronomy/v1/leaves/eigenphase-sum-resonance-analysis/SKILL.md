---
name: eigenphase-sum-resonance-analysis
description: Use when a coupled-channel scattering calculation must locate a Feshbach
  or shape resonance via its signature in the eigenphase sum, e.g. two coupled Lennard-Jones
  channels with a closed upper channel shifted by ~100 cm-1.
when_to_use_negative:
- The target is bound-state energies rather than scattering resonances — use BOUND
  instead of the molscat eigenphase-sum scan.
- The goal is to locate a magnetic-field value at which a bound state exists at a
  specified energy — FIELD is designed for that, not an energy scan of the eigenphase
  sum.
- The collision system is reactive; MOLSCAT performs only non-reactive quantum scattering
  calculations (additional collision types require plug-in routines).
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: molscat
  role: Performs the close-coupling scattering calculation and produces the S matrices/eigenphase
    sum whose scan reveals the resonance
  repo: https://github.com/molscat/molscat
- name: LAPACK
  role: Linear algebra routines used by molscat during the coupled-channel propagation
  repo: https://github.com/molscat/molscat
- name: BLAS
  role: Basic linear algebra subroutines underlying the LAPACK routines used in the
    calculation
  repo: https://github.com/molscat/molscat
- name: GNUmakefile
  role: Build system that must be adapted (LIBS/LIBUTILS, Compiler) to compile the
    two-channel molscat variant via a make rule such as 'make molscat-H2_H2'
  repo: https://github.com/molscat/molscat
- name: RESFIT
  role: Post-processor for fitting resonance parameters from molscat output
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:d4aedd455ec5ad19092ffc282a597f352922551260bde4717b69024adb09ad37
  source_papers:
  - doi: 10.48550/arxiv.1903.06755
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/molscat/molscat
  tools:
  - BLAS
  - GNUmakefile
  - LAPACK
  - RESFIT
  - molscat
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# eigenphase-sum-resonance-analysis

## Summary

Locate and characterize scattering resonances by scanning the eigenphase sum as a function of collision energy (or external field) using MOLSCAT close-coupling calculations, as demonstrated by the two-channel Lennard-Jones Feshbach resonance example (molscat-2chan-LJ) where a closed-channel bound state produces a resonance just below 42.84 cm-1.

## When to use

Use when a coupled-channel scattering calculation must locate a Feshbach or shape resonance via its signature in the eigenphase sum, e.g. two coupled Lennard-Jones channels with a closed upper channel shifted by ~100 cm-1.

## When NOT to use

- The target is bound-state energies rather than scattering resonances — use BOUND instead of the molscat eigenphase-sum scan.
- The goal is to locate a magnetic-field value at which a bound state exists at a specified energy — FIELD is designed for that, not an energy scan of the eigenphase sum.
- The collision system is reactive; MOLSCAT performs only non-reactive quantum scattering calculations (additional collision types require plug-in routines).

## Inputs

- molscat namelist input file (e.g. molscat-2chan-LJ.input with &INPUT, &BASIS, &POTL blocks) defining two Lennard-Jones channels separated by 100 cm-1 with a coupling potential 10x weaker than the channel potentials
- GNUmakefile adapted with LIBS (optimised BLAS/LAPACK) or LIBUTILS (Netlib object files) and Compiler (gfortran)
- Reference output files from examples/output/ for comparison

## Outputs

- Eigenphase sum as a function of collision energy near 42.84 cm-1 showing the Feshbach resonance feature
- Resonance position (just below 42.84 cm-1) identified from the eigenphase-sum signature
- molscat output file matching the reference output in examples/output/

## How to apply

Adapt GNUmakefile for the local environment (set LIBS to optimised BLAS/LAPACK libraries or LIBUTILS to Netlib object files, set Compiler to gfortran) and build the two-channel variant, e.g. with `make molscat-H2_H2` following the same GNUmakefile rule pattern. Run the executable on the example input molscat-2chan-LJ.input from examples/input/, whose namelist blocks (&INPUT, &BASIS, &POTL) define two Lennard-Jones channels separated by 100 cm-1 and coupled by an interaction potential 10x weaker than the channel potentials. Perform a close-coupling calculation (no dynamical approximations) that loops over total angular momentum (partial wave) and records the eigenphase sum versus collision energy across the region near 42.84 cm-1. Identify the resonance position from the eigenphase-sum signature — MOLSCAT can 'converge on and characterise scattering resonances or predissociating / quasibound states of the collision complex through their signature in the eigenphase sum' — and confirm it lies just below 42.84 cm-1, arising from a bound state in the closed upper channel shifted by the 100 cm-1 threshold separation. Validate by diffing the generated output against the reference file in examples/output/.

## Related tools

- **molscat** (Performs the close-coupling scattering calculation and produces the S matrices/eigenphase sum whose scan reveals the resonance) — https://github.com/molscat/molscat
- **LAPACK** (Linear algebra routines used by molscat during the coupled-channel propagation) — https://github.com/molscat/molscat
- **BLAS** (Basic linear algebra subroutines underlying the LAPACK routines used in the calculation) — https://github.com/molscat/molscat
- **GNUmakefile** (Build system that must be adapted (LIBS/LIBUTILS, Compiler) to compile the two-channel molscat variant via a make rule such as 'make molscat-H2_H2') — https://github.com/molscat/molscat
- **RESFIT** (Post-processor for fitting resonance parameters from molscat output)

## Examples

```
make molscat-H2_H2 && ./molscat-H2_H2 < examples/input/molscat-2chan-LJ.input
```

## Evaluation signals

- The eigenphase-sum scan exhibits a Feshbach resonance feature located just below 42.84 cm-1
- The generated output file matches the associated reference output file in examples/output/ (the acceptance criterion cited in section 14.1.12)
- The two-channel model resolves with the closed upper channel shifted by the 100 cm-1 threshold separation and a coupling potential 10x weaker than the channel potentials
- The build succeeds with the adapted GNUmakefile (LIBS/LIBUTILS, Compiler=gfortran) using the make-rule pattern 'make molscat-H2_H2'

## Limitations

- The programs use LAPACK routines, which need optimised BLAS/LAPACK libraries in LIBS or Netlib object files in LIBUTILS; a missing library setup will break the build.
- The default optimisation level is 0 with traceback switched on; performance may require increasing optimisation after correctness is verified.
- The documented resonance position is specific to this two-channel Lennard-Jones model (100 cm-1 threshold separation, coupling 10x weaker); other systems require their own scans.

## Evidence
