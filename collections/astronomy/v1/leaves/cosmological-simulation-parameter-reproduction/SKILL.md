---
name: cosmological-simulation-parameter-reproduction
description: Reference for reproducing a published cosmological simulation (e.g. the
  512 Mpc/h, N=1024^3 CONCEPT 1.0 vs GADGET-4 comparison) when the goal is to recreate
  runs from released parameter files and validate code outputs against reported agreement
  levels.
when_to_use_negative:
- The goal is only to measure power spectra from existing snapshots rather than re-running
  and reproducing published simulations from their parameter files.
- The target hardware/code configuration differs materially from the published setup
  (e.g. comparing against GADGET-2 rather than GADGET-4, or different box size/particle
  count), so the paper's agreement thresholds do not transfer.
- Only a GADGET-format snapshot with limited output fidelity is available, since the
  article notes 'GADGET format (limited)' as an output restriction.
tools:
- name: CONCEPT 1.0
  role: primary simulation code run with the published reference parameter file; also
    used for FFTW-based power-spectrum measurement of snapshots
  repo: github.com/jmd-dk/concept
- name: GADGET-4
  role: comparison code run on identical initial conditions with TreePM and FMM-PM
    force solvers at specified multipole orders
- name: class
  role: linear Einstein-Boltzmann code used as the reference for initial conditions
    / linear behaviour
  repo: github.com/lesgourg/class_public
- name: FFTW
  role: MPI-parallelised real 3D in-place transforms used in CONCEPT's power-spectrum
    estimation pipeline
- name: OpenMPI
  role: MPI runtime used to launch CONCEPT simulations
- name: NumPy
  role: post-processing of power-spectrum measurements and ratio tabulation
  repo: github.com/numpy/numpy
- name: Docker
  role: alternative reproducible installation route via published CONCEPT 1.0 images
  repo: hub.docker.com/r/jmddk/concept
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/mnras/stac568
    title: The cosmological simulation code CO<i>N</i>CEPT 1.0
  source_content_hash: sha256:03c9ee0cac289da5260bcf1f6e5d01cb1fb74ec0a464fb302262d7084e50cc9f
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - CONCEPT 1.0
  - Docker
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
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# cosmological-simulation-parameter-reproduction

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Reproduce published cosmological N-body simulation runs by retrieving the authors' released simulation parameter files (e.g. from github.com/Aarhus Cosmology/concept1.0-data) and rerunning codes such as CONCEPT 1.0 and GADGET-4 on identical initial conditions for cross-code validation.

## When to use

Reference for reproducing a published cosmological simulation (e.g. the 512 Mpc/h, N=1024^3 CONCEPT 1.0 vs GADGET-4 comparison) when the goal is to recreate runs from released parameter files and validate code outputs against reported agreement levels.

## When NOT to use

- The goal is only to measure power spectra from existing snapshots rather than re-running and reproducing published simulations from their parameter files.
- The target hardware/code configuration differs materially from the published setup (e.g. comparing against GADGET-2 rather than GADGET-4, or different box size/particle count), so the paper's agreement thresholds do not transfer.
- Only a GADGET-format snapshot with limited output fidelity is available, since the article notes 'GADGET format (limited)' as an output restriction.

## Inputs

- simulation parameter files from github.com/Aarhus Cosmology/concept1.0-data
- identical initial conditions for N-body runs (512 Mpc/h box, N=1024^3 particles)
- particle snapshot outputs from CONCEPT 1.0 and GADGET-4 runs

## Outputs

- re-run simulation snapshots per force-solver configuration (TreePM order 2/5, FMM-PM order 5 with/without randomised displacements)
- relative matter power spectra ratios P_CONCEPT/P_GADGET-4 versus wavenumber (e.g. relative_spectra_cond_concept_vs_gadget4.csv)
- CONCEPT test-suite log files (basic test, GADGET test, full suite)

## How to apply

First retrieve the simulation parameter files for the target setup from the authors' public data repository (github.com/Aarhus Cosmology/concept1.0-data), which exists precisely so simulations can be recreated. Install the full dependency stack in the documented order (zlib, GSL, Perl, MPI, HDF5, FFTW, ncurses, OpenSSL, libffi, OpenBLAS, Python, CONCEPT, CLASS, FFTW 2, GADGET) or use the provided Docker images, then verify the build by running CONCEPT's test suites ('basic', GADGET test, full suite) launched through OpenMPI. Run CONCEPT 1.0 with the published reference parameter file (512 Mpc/h box, N=1024^3 particles) and run GADGET-4 on identical initial conditions in the configurations specified by the paper (e.g. TreePM at multipole orders 2 and 5; FMM-PM order 5 with and without randomised displacements). Measure matter power spectra from particle snapshots with CONCEPT's FFTW-based pipeline and NumPy post-processing, form ratios such as P_CONCEPT/P_GADGET-4, and judge success against the paper's thresholds: agreement well below the per mille level at large and intermediate scales, of order 1% at k ~ 13 h/Mpc, and reproduction of configuration-specific effects such as the z=10 FMM-PM power drop removed by randomised displacements.

## Related tools

- **CONCEPT 1.0** (primary simulation code run with the published reference parameter file; also used for FFTW-based power-spectrum measurement of snapshots) — github.com/jmd-dk/concept
- **GADGET-4** (comparison code run on identical initial conditions with TreePM and FMM-PM force solvers at specified multipole orders)
- **class** (linear Einstein-Boltzmann code used as the reference for initial conditions / linear behaviour) — github.com/lesgourg/class_public
- **FFTW** (MPI-parallelised real 3D in-place transforms used in CONCEPT's power-spectrum estimation pipeline)
- **OpenMPI** (MPI runtime used to launch CONCEPT simulations)
- **NumPy** (post-processing of power-spectrum measurements and ratio tabulation) — github.com/numpy/numpy
- **Docker** (alternative reproducible installation route via published CONCEPT 1.0 images) — hub.docker.com/r/jmddk/concept

## Examples

```
"${concept}" -t "basic" 2>&1 | tee "${concept_test_log}"
```

## Evaluation signals

- CONCEPT basic/GADGET/full test suites pass after installation ('${concept}' -t 'basic' completes successfully).
- Order-5 configurations agree with CONCEPT well below the per mille level at large and intermediate scales.
- Agreement at the smallest probed scales (k ~ 13 h/Mpc) is of order 1%, matching the published Figure 6 behaviour.
- The un-randomised FMM-PM run reproduces the several per mille power drop at z=10, which disappears when randomised displacements are enabled.
- Relative spectra ratio-versus-wavenumber tables (e.g. relative_spectra_cond_concept_vs_gadget4.csv) are produced for every force-solver configuration.

## Limitations

- CONCEPT 1.0 is several times faster than GADGET-2 only for low to medium resolutions (k_Nyquist < 5 Mpc/h) and slower at higher resolution, affecting run-time expectations.
- FFTW slab decomposition limits the number of CPU cores participating in the long-range force computation, and load imbalance degrades performance in strongly clustered systems.
- P3M gravity in CONCEPT is noted as inefficient, so it is not a viable alternative solver for reproduction runs.
- GADGET snapshot output is limited, which constrains downstream measurement options.

## Evidence

- [abstract] simulation parameter files for recreating the simulations producing this data — are made publicly available at github.com/Aarhus Cosmology/concept1.0-data: [evidence span withheld — anchor status: not_found]
- [abstract] agreement thresholds: "At large and intermediate scales the codes agree to well below the per mille level, while the agreement at the smallest scales probed (k ∼ 13 h/Mpc) is of the order of 1 %"
- [methods] install order: "The programs which will be installed into the specified installation
directory consist of the following and will be installed in order:"
- [methods] test suite invocation: ""${concept}" -t "basic" 2>&1 | tee "${concept_test_log}""
- [readme] README portability: "free and open-source simulation code for cosmological structure formation. The code should run on any Linux system,"
