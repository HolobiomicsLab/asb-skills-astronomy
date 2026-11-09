---
name: numerical-agreement-tolerance-verification
description: Use when two N-body or cosmology codes are run on identical initial conditions
  and you must quantify their agreement by forming relative power spectra P_A/P_B
  versus wavenumber and checking per-mille-level tolerances.
when_to_use_negative:
- Input is a single code's power spectrum with no matched comparison run — there is
  no ratio to form
- The codes were run on different boxes, resolutions, or initial conditions — relative
  spectra then mix physical differences with numerical ones
- Goal is linear-theory validation against an Einstein-Boltzmann code such as CLASS
  rather than cross-code agreement between N-body codes
tools:
- name: CONCEPT 1.0
  role: Reference simulation code; its FFTW-based pipeline measures power spectra
    from snapshots; launched through OpenMPI
  repo: github.com/jmd-dk/concept
- name: GADGET-4
  role: Comparison code run with TreePM (orders 2 and 5) and FMM-PM (order 5, with/without
    randomised displacements) on identical initial conditions
- name: FFTW
  role: MPI-parallelised, real, 3D in-place FFTs underlying CONCEPT's power-spectrum
    measurement
- name: NumPy
  role: Post-processing of measured spectra and computation of relative ratios
  repo: github.com/numpy/numpy
- name: OpenMPI
  role: Used to launch the CONCEPT 1.0 runs
- name: class
  role: Linear Einstein-Boltzmann reference for the initial power spectrum the simulations
    are designed to agree with
  repo: github.com/lesgourg/class_public
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/mnras/stac568
    title: The cosmological simulation code CO<i>N</i>CEPT 1.0
  source_content_hash: sha256:c7ae2c173886477ea6c16e434560ac21ae749458a0b892ae7cf42922c7641de4
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# numerical-agreement-tolerance-verification

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Verifies that independent simulation codes (e.g. CONCEPT 1.0 vs GADGET-2/GADGET-4) reproduce the same matter power spectrum within stated numerical tolerances — well below the per mille at large/intermediate scales and ~1% at k ~ 13 h/Mpc — by forming relative spectra from matched runs on identical initial conditions.

## When to use

Use when two N-body or cosmology codes are run on identical initial conditions and you must quantify their agreement by forming relative power spectra P_A/P_B versus wavenumber and checking per-mille-level tolerances.

## When NOT to use

- Input is a single code's power spectrum with no matched comparison run — there is no ratio to form
- The codes were run on different boxes, resolutions, or initial conditions — relative spectra then mix physical differences with numerical ones
- Goal is linear-theory validation against an Einstein-Boltzmann code such as CLASS rather than cross-code agreement between N-body codes

## Inputs

- Particle snapshots at z=0 and z=10 from each code/solver configuration (TreePM order 2, order 5; FMM-PM order 5 with/without randomised displacements)
- Simulation parameter files for the comparison setup (github.com/Aarhus Cosmology/concept1.0-data)
- Identical initial conditions in a 512 Mpc/h box with N=1024^3 particles
- CONCEPT's FFTW-based power-spectrum measurement pipeline with NumPy post-processing

## Outputs

- Matter power spectra per code and force-solver configuration
- Relative spectra P_CONCEPT/P_GADGET-4 versus wavenumber, tabulated in relative_spectra_cond_concept_vs_gadget4.csv
- Pass/fail assessment against per-mille (large/intermediate k) and ~1% (k ~ 13 h/Mpc) tolerance targets

## How to apply

Run each code (CONCEPT 1.0 via OpenMPI, GADGET-4 with TreePM at multipole orders 2 and 5 and FMM-PM at order 5, with and without randomised displacements) on identical initial conditions — here a 512 Mpc/h box with N=1024^3 particles using the parameter files from github.com/Aarhus Cosmology/concept1.0-data. Measure each matter power spectrum from particle snapshots with CONCEPT's FFTW-based pipeline and NumPy post-processing, then tabulate the ratio P_CONCEPT/P_GADGET-4 versus wavenumber per solver configuration. Judge correctness against the published tolerances: agreement well below the per mille at large and intermediate scales and of order 1% at the smallest scales probed (k ~ 13 h/Mpc). Treat configuration-specific deviations as physical signals, not bugs: the order-5 FMM-PM run must show a several-per-mille power drop at z=10 that enabling randomised displacements removes, while late-time (z=0) behaviour stays practically unchanged.

## Related tools

- **CONCEPT 1.0** (Reference simulation code; its FFTW-based pipeline measures power spectra from snapshots; launched through OpenMPI) — github.com/jmd-dk/concept
- **GADGET-4** (Comparison code run with TreePM (orders 2 and 5) and FMM-PM (order 5, with/without randomised displacements) on identical initial conditions)
- **FFTW** (MPI-parallelised, real, 3D in-place FFTs underlying CONCEPT's power-spectrum measurement)
- **NumPy** (Post-processing of measured spectra and computation of relative ratios) — github.com/numpy/numpy
- **OpenMPI** (Used to launch the CONCEPT 1.0 runs)
- **class** (Linear Einstein-Boltzmann reference for the initial power spectrum the simulations are designed to agree with) — github.com/lesgourg/class_public

## Examples

```
mpirun -np 1024 concept parameters_1024_concept_vs_gadget4.ini
```

## Evaluation signals

- Relative spectra P_CONCEPT/P_GADGET-4 lie well below the per mille level at large and intermediate scales for all order-5 configurations
- Disagreement at the smallest scales probed (k ~ 13 h/Mpc) is of the order of 1%, not larger
- The un-randomised FMM-PM order-5 run shows a several-per-mille power suppression relative to CONCEPT at z=10, and enabling randomised displacements removes this early-time drop
- Enabling randomised displacements leaves the late-time (z=0) relative behaviour practically the same
- Ratio table relative_spectra_cond_concept_vs_gadget4.csv has one curve per solver configuration with ratios defined on the same k grid

## Limitations

- Per-mille agreement holds only at large and intermediate scales; expect ~1% discrepancies at the smallest scales probed (k ~ 13 h/Mpc), so these modes cannot be used as a tight agreement check
- Force-solver configuration changes have physical signatures (randomised displacements fix the early-time FMM-PM power drop), so a tolerance failure may indicate solver settings rather than an implementation error
- CONCEPT uses the standard slab-decomposition of FFTW, which limits the number of CPU cores participating in the long-range force computation, constraining how the comparison runs can be scaled

## Evidence

- [abstract] FFTW measurement pipeline: [evidence span withheld — anchor status: not_found]
- [abstract] published parameter files for reproducing the comparison data: [evidence span withheld — anchor status: not_found]
