---
name: force-solver-configuration-treepm-fmm
description: Use when validating two cosmological N-body codes (e.g. CONCEPT 1.0 vs
  gadget-4) against each other on the same initial conditions, or when diagnosing
  a scale- or time-dependent power discrepancy attributable to the gravitational force
  solver configuration.
when_to_use_negative:
- You need to compare performance/runtime rather than power-spectrum accuracy — CONCEPT
  is only several times faster than GADGET-2 for low to medium resolutions (k_Nyquist
  < 5 Mpc/h) and slower at higher resolution, so timing claims need the separate benchmarking
  protocol.
- The runs were not started from identical initial conditions or do not share box
  size/resolution — relative ratios then mix differences in ICs and sampling with
  solver differences.
- Your question concerns linear-theory predictions, where agreement with class, not
  N-body force solvers, is the reference.
edam_operation: http://edamontology.org/operation_0242
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: CONCEPT 1.0
  role: Runs the reference N-body simulation and provides the FFTW-based power-spectrum
    measurement pipeline for all runs
  repo: github.com/jmd-dk/concept
- name: GADGET-4
  role: 'Provides the comparison force solvers: TreePM (orders 2 and 5) and FMM-PM
    (order 5) with optional randomised displacements'
- name: class
  role: Linear Einstein-Boltzmann reference the simulation power spectra are designed
    to agree with
  repo: github.com/lesgourg/class_public
- name: NumPy
  role: Post-processing of measured spectra and formation of the relative-ratio table
  repo: github.com/numpy/numpy
- name: FFTW
  role: MPI-parallelised, real, 3D in-place transformations underlying CONCEPT's spectrum
    measurement
- name: OpenMPI
  role: Launches and parallelises CONCEPT 1.0 across cores
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/mnras/stac568
    title: The cosmological simulation code CO<i>N</i>CEPT 1.0
  source_content_hash: sha256:3e21ce3eb5ea09acb218f3a3055c929e4898afd5f5ebef7d1d83a13d6f74f7d5
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - CONCEPT 1.0
  - FFTW
  - GADGET-4
  - NumPy
  - OpenMPI
  - Python
  - class
  edam_operation: http://edamontology.org/operation_0242
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Force-solver configuration for TreePM/FMM-PM cross-code power-spectrum comparison

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Configure CONCEPT 1.0 and gadget-4 N-body runs on identical initial conditions with TreePM (multipole orders 2 and 5) and FMM-PM (order 5, with and without randomised displacements), then form relative matter power spectra P_CONCEPT/P_gadget-4 to expose solver-dependent biases such as the several per mille early-time (z=10) FMM-PM power drop.

## When to use

Use when validating two cosmological N-body codes (e.g. CONCEPT 1.0 vs gadget-4) against each other on the same initial conditions, or when diagnosing a scale- or time-dependent power discrepancy attributable to the gravitational force solver configuration.

## When NOT to use

- You need to compare performance/runtime rather than power-spectrum accuracy — CONCEPT is only several times faster than GADGET-2 for low to medium resolutions (k_Nyquist < 5 Mpc/h) and slower at higher resolution, so timing claims need the separate benchmarking protocol.
- The runs were not started from identical initial conditions or do not share box size/resolution — relative ratios then mix differences in ICs and sampling with solver differences.
- Your question concerns linear-theory predictions, where agreement with class, not N-body force solvers, is the reference.

## Inputs

- CONCEPT 1.0 reference parameter file for the 512 Mpc/h, N=1024^3 box (from github.com/Aarhus Cosmology/concept1.0-data)
- gadget-4 parameter files configured for TreePM (orders 2, 5) and FMM-PM (order 5, with/without randomised displacements)
- Identical initial conditions shared by both codes
- Particle snapshots at z=10 (early time) and z=0 (late time)

## Outputs

- Matter power spectra P(k) measured from each run's particle snapshots via CONCEPT's FFTW pipeline
- relative_spectra_cond_concept_vs_gadget4.csv tabulating P_CONCEPT/P_gadget-4 versus wavenumber for each force-solver configuration

## How to apply

Retrieve the reference simulation parameter files from github.com/Aarhus Cosmology/concept1.0-data and run CONCEPT 1.0 (launched through OpenMPI) on the 512 Mpc/h box with N=1024^3 particles. Run gadget-4 on identical initial conditions in four force-solver configurations: TreePM at multipole expansion orders 2 and 5, and FMM-PM at order 5 both with and without randomised displacements. Estimate the matter power spectrum of each run from particle snapshots using CONCEPT's FFTW-based measurement pipeline with NumPy post-processing, form the ratio P_CONCEPT/P_gadget-4 versus wavenumber, and tabulate it in relative_spectra_cond_concept_vs_gadget4.csv. The decision point is the randomised-displacement toggle for FMM-PM: enabling it removes the early-time suppression without changing late-time behaviour, which is what identifies the discrepancy as solver-side rather than physical.

## Related tools

- **CONCEPT 1.0** (Runs the reference N-body simulation and provides the FFTW-based power-spectrum measurement pipeline for all runs) — github.com/jmd-dk/concept
- **GADGET-4** (Provides the comparison force solvers: TreePM (orders 2 and 5) and FMM-PM (order 5) with optional randomised displacements)
- **class** (Linear Einstein-Boltzmann reference the simulation power spectra are designed to agree with) — github.com/lesgourg/class_public
- **NumPy** (Post-processing of measured spectra and formation of the relative-ratio table) — github.com/numpy/numpy
- **FFTW** (MPI-parallelised, real, 3D in-place transformations underlying CONCEPT's spectrum measurement)
- **OpenMPI** (Launches and parallelises CONCEPT 1.0 across cores)

## Examples

```
mpirun -np 1024 concept params_512Mpc_1024.concept
```

## Evaluation signals

- At large and intermediate scales, order-5 TreePM and FMM-PM configurations agree with CONCEPT well below the per mille level in P_CONCEPT/P_gadget-4.
- At the smallest scales probed (k ~ 13 h/Mpc), the ratio is of order 1%, not worse.
- The un-randomised order-5 FMM-PM run shows a several per mille power drop relative to CONCEPT at z=10; this suppression must be absent at z=0 and removed when randomised displacements are enabled.
- Enabling randomised displacements leaves the late-time (z=0) ratio practically unchanged, confirming the fix is early-time-specific.
- The ratio table is written as relative_spectra_cond_concept_vs_gadget4.csv with ratio versus wavenumber columns for each solver configuration.

## Limitations

- P3M gravity is noted as an available but inefficient option in CONCEPT, so it is not a suitable comparison configuration.
- CONCEPT uses the standard slab-decomposition of FFTW, which limits the number of CPU cores participating in the long-range force computation.
- Agreement between codes degrades to ~1% at the smallest scales (k ~ 13 h/Mpc), so per-mille-level conclusions are only valid at large and intermediate scales.
- GADGET-format snapshot output is described as limited, which constrains downstream snapshot handling.

## Evidence

- [other] FMM-PM early-time discrepancy and fix via randomised displacements: [evidence span withheld — anchor status: not_found]
- [abstract] CONCEPT's FFTW measurement engine: [evidence span withheld — anchor status: not_found]
