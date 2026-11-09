---
name: cross-code-n-body-power-spectrum-comparison
description: Use when two or more N-body codes must be validated against each other
  via relative power spectra P_codeA/P_codeB, especially when diagnosing force-solver
  differences (multipole order, randomised displacements) at early vs late redshifts.
when_to_use_negative:
- Validating a single code against the linear theory of an Einstein-Boltzmann solver
  such as CLASS rather than against another N-body code
- Benchmarking runtime performance or scaling rather than spectral accuracy — CONCEPT
  is several times faster than GADGET-2 only for k_Nyquist < 5 Mpc/h and slower at
  higher resolution
- Cases where initial conditions are not shared between the codes, since ratios then
  conflate IC differences with force-solver differences
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3452
tools:
- name: CONCEPT 1.0
  role: Runs one side of the comparison and supplies the FFTW-based power-spectrum
    measurement pipeline for all snapshots
  repo: https://github.com/jmd-dk/concept
- name: GADGET-4
  role: Runs the reference TreePM (orders 2 and 5) and FMM-PM (order 5, with/without
    randomised displacements) simulations on identical initial conditions
- name: FFTW
  role: MPI-parallelised real 3D in-place transforms underpinning CONCEPT's power-spectrum
    measurement
- name: NumPy
  role: Post-processing of measured spectra into relative ratios
  repo: https://github.com/numpy/numpy
- name: OpenMPI
  role: Launches the parallel CONCEPT simulation runs
- name: class
  role: Linear Einstein-Boltzmann reference that the simulation initial conditions
    are designed to agree with
  repo: https://github.com/lesgourg/class_public
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/mnras/stac568
    title: The cosmological simulation code CO<i>N</i>CEPT 1.0
  source_content_hash: sha256:157c05f93b8de3bb63c9fdac2b17ca34044f35371e092ebe4d76d2eeef29bac7
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/jmd-dk/concept
  tools:
  - CONCEPT 1.0
  - FFTW
  - GADGET-4
  - NumPy
  - OpenMPI
  - Python
  - class
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# cross-code-n-body-power-spectrum-comparison

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Compares matter power spectra from CONCEPT 1.0 and GADGET-4 (TreePM and FMM-PM solvers) on identical initial conditions in a 512 Mpc/h box with N=1024^3 particles, quantifying code-to-code agreement and diagnosing solver-specific discrepancies such as the early-time (z=10) FMM-PM power suppression.

## When to use

Use when two or more N-body codes must be validated against each other via relative power spectra P_codeA/P_codeB, especially when diagnosing force-solver differences (multipole order, randomised displacements) at early vs late redshifts.

## When NOT to use

- Validating a single code against the linear theory of an Einstein-Boltzmann solver such as CLASS rather than against another N-body code
- Benchmarking runtime performance or scaling rather than spectral accuracy — CONCEPT is several times faster than GADGET-2 only for k_Nyquist < 5 Mpc/h and slower at higher resolution
- Cases where initial conditions are not shared between the codes, since ratios then conflate IC differences with force-solver differences

## Inputs

- Simulation parameter files for the CONCEPT-vs-GADGET-4 comparison setup (from github.com/Aarhus Cosmology/concept1.0-data)
- Particle snapshots from CONCEPT 1.0 and GADGET-4 runs on identical initial conditions (512 Mpc/h box, N=1024^3, z=10 and z=0)

## Outputs

- Matter power spectra measured from each run's snapshots
- Relative spectra P_CONCEPT/P_gadget-4 per force-solver configuration, tabulated as ratio versus wavenumber (e.g. relative_spectra_cond_concept_vs_gadget4.csv)

## How to apply

Retrieve the simulation parameter files from github.com/Aarhus Cosmology/concept1.0-data, run CONCEPT 1.0 (launched through OpenMPI) on the 512 Mpc/h box with N=1024^3 particles using the published reference parameter file, and run GADGET-4 on identical initial conditions with TreePM at multipole orders 2 and 5 and FMM-PM at order 5 with and without randomised displacements. Measure each run's matter power spectrum from particle snapshots with CONCEPT's FFTW-based measurement pipeline and post-process with NumPy. Form the relative spectra P_CONCEPT/P_gadget-4 per force-solver configuration and tabulate ratio versus wavenumber (e.g. relative_spectra_cond_concept_vs_gadget4.csv). Interpret: a several per mille order-5 FMM-PM drop at z=10 that randomised displacements remove is a solver artefact, not a physical difference.

## Related tools

- **CONCEPT 1.0** (Runs one side of the comparison and supplies the FFTW-based power-spectrum measurement pipeline for all snapshots) — https://github.com/jmd-dk/concept
- **GADGET-4** (Runs the reference TreePM (orders 2 and 5) and FMM-PM (order 5, with/without randomised displacements) simulations on identical initial conditions)
- **FFTW** (MPI-parallelised real 3D in-place transforms underpinning CONCEPT's power-spectrum measurement)
- **NumPy** (Post-processing of measured spectra into relative ratios) — https://github.com/numpy/numpy
- **OpenMPI** (Launches the parallel CONCEPT simulation runs)
- **class** (Linear Einstein-Boltzmann reference that the simulation initial conditions are designed to agree with) — https://github.com/lesgourg/class_public

## Examples

```
mpirun -np 256 concept --parameters reference.params; python measure_powerspectra.py --snapshots run_dir/snap_*  # ratio-vs-k table: relative_spectra_cond_concept_vs_gadget4.csv
```

## Evaluation signals

- Order-5 configurations (TreePM and randomised FMM-PM) agree with CONCEPT well below the per mille level at large and intermediate scales
- At the smallest scales probed (k ~ 13 h/Mpc), agreement of the order of 1% is expected — larger deviations flag a pipeline error
- The un-randomised order-5 FMM-PM run must exhibit a several per mille power suppression relative to CONCEPT at z=10, which enabling randomised displacements removes
- The z=0 relative spectra must remain practically unchanged between the randomised and un-randomised FMM-PM variants
- The relative_spectra CSV must contain ratio-vs-wavenumber rows for every force-solver configuration (TreePM order 2, TreePM order 5, FMM-PM order 5 ± randomised displacements)

## Limitations

- Order-5 FMM-PM in GADGET-4 shows a several per mille early-time (z=10) power drop unless randomised displacements are enabled
- Power-spectrum measurement itself is resolution-limited; agreement degrades to ~1% only at k ~ 13 h/Mpc, so per mille claims hold only at large and intermediate scales
- CONCEPT's FFTW slab-decomposition limits the number of CPU cores usable for the long-range force computation, which constrains how the large N=1024^3 comparison runs are launched

## Evidence

- [full_text] Order-5 FMM-PM in gadget-4 shows a several per mille drop in power relative to concept 1.0 at z = 10, and enabling randomised displacements fixes this early-time behaviour: [evidence span withheld — anchor status: not_found]
- [abstract] Parameter files for reproducing simulations are publicly hosted: [evidence span withheld — anchor status: not_found]
- [abstract] CONCEPT run through OpenMPI and uses FFTW MPI-parallel real 3D in-place transforms: [evidence span withheld — anchor status: not_found]
- [abstract] Slab decomposition limits cores for long-range force computation: [evidence span withheld — anchor status: not_found]
