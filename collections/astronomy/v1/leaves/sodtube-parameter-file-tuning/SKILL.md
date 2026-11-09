---
name: sodtube-parameter-file-tuning
description: 'Use when reproducing or scaling the FleCSPH Sod shock tube validation
  in 1D/2D/3D: you have a compiled FleCSPH build and need to set up or adjust the
  simulation parameter file and initial-condition particle count.'
when_to_use_negative:
- The target problem is not a Sod shock tube (e.g. Noh shock, Sedov blast, airfoil
  flow) — different generators and parameter files are required
- The goal is quantitative error metrics against the Riemann solution — the paper
  only supports qualitative visual/pointwise profile agreement, so this skill cannot
  supply numeric acceptance thresholds
- FleCSPH has not been built — the parameter file alone is useless without the compiled
  sodtube generator and hydro driver executables
tools:
- name: FleCSPH
  role: SPH simulation application providing the sodtube generators and hydro drivers
    exercised by the tuned parameter file
  repo: https://github.com/laristra/flecsph
- name: FleCSI
  role: Compile-time configurable parallel framework that FleCSPH must be built against
    (MPI backend, C++17, METIS/coloring enabled)
  repo: https://github.com/laristra/flecsi
- name: H5part
  role: Output format for the particle snapshots produced by the hydro driver
- name: Spack
  role: 'Alternative installation route: spack install flecsph and spack load flecsph
    provides the generators and drivers'
  repo: https://github.com/spack/spack
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.softx.2020.100602
    title: 'FleCSPH: The next generation FleCSIble parallel computational infrastructure
      for smoothed particle hydrodynamics'
  source_content_hash: sha256:7ec83e64b9640a09fa7aab95d2f4993448e8cd180aedf3e9be23de5a06c86ca7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/laristra/flecsph
  tools:
  - FleCSI
  - FleCSPH
  - H5part
  - MPI
  - Spack
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# sodtube-parameter-file-tuning

## Summary

Configure and tune FleCSPH Sod shock tube simulations by copying a pre-configured parameter file from the data subdirectory (e.g. sodtube_t1_n1000.par), editing particle count and settings, generating initial conditions with a compiled sodtube generator, and running the hydro driver to produce H5part snapshots for comparison against the classical Riemann solution.

## When to use

Use when reproducing or scaling the FleCSPH Sod shock tube validation in 1D/2D/3D: you have a compiled FleCSPH build and need to set up or adjust the simulation parameter file and initial-condition particle count.

## When NOT to use

- The target problem is not a Sod shock tube (e.g. Noh shock, Sedov blast, airfoil flow) — different generators and parameter files are required
- The goal is quantitative error metrics against the Riemann solution — the paper only supports qualitative visual/pointwise profile agreement, so this skill cannot supply numeric acceptance thresholds
- FleCSPH has not been built — the parameter file alone is useless without the compiled sodtube generator and hydro driver executables

## Inputs

- Pre-configured parameter file from the data subdirectory (e.g. data/sodtube_t1_n1000.par)
- Edited sodtube.par parameter file with adjusted particle count
- Compiled FleCSPH generator binary (sodtube_1d_generator built with -DEXT_GDIMENSION=1) and hydro driver (hydro_1d)

## Outputs

- Initial-condition particle data generated from sodtube.par
- Final particle snapshot in H5part format from app/driver/hydro_1d
- Density and velocity profile tables compared against the classical Sod Riemann solution

## How to apply

Build FleCSPH from the laristra repository on the stable/flecsph branch with FleCSI configured for the MPI backend (-DFLECSI_RUNTIME_MODEL=mpi), C++17, METIS/ParMETIS and coloring enabled, and OpenMP off. Copy the pre-configured example file from the data subdirectory (cp ../data/sodtube_t1_n1000.par sodtube.par) and edit it to adjust the particle count and simulation settings. Compile the dimension-specific generator with -DEXT_GDIMENSION=1 for a 1D test and run app/id_generators/sodtube_1d_generator sodtube.par to create the initial particle data. Run the evolution driver app/driver/hydro_1d sodtube.par, which writes particle snapshots in H5part format. Validation is qualitative: the final density and velocity profiles are accepted if they visually/pointwise match the classical Sod Riemann solution profiles, since the paper reports no quantitative error threshold.

## Related tools

- **FleCSPH** (SPH simulation application providing the sodtube generators and hydro drivers exercised by the tuned parameter file) — https://github.com/laristra/flecsph
- **FleCSI** (Compile-time configurable parallel framework that FleCSPH must be built against (MPI backend, C++17, METIS/coloring enabled)) — https://github.com/laristra/flecsi
- **H5part** (Output format for the particle snapshots produced by the hydro driver)
- **Spack** (Alternative installation route: spack install flecsph and spack load flecsph provides the generators and drivers) — https://github.com/spack/spack

## Examples

```
cp ../data/sodtube_t1_n1000.par sodtube.par && app/id_generators/sodtube_1d_generator sodtube.par && app/driver/hydro_1d sodtube.par
```

## Evaluation signals

- The generator runs without error on the edited sodtube.par and produces initial particle data in the expected format
- The hydro driver completes and writes a final H5part snapshot
- Extracted density and velocity profiles from the final snapshot qualitatively (visually/pointwise) match the classical Sod Riemann solution profiles
- The particle count in the generated initial data reflects the count edited into the parameter file
- Executables are obtainable either from the built tree or after 'spack load flecsph' (sodtube_{1-2-3}d_generator, hydro_{1-2-3}d)

## Limitations

- The paper reports no quantitative error threshold: acceptance of the density and velocity profiles is judged by visual/pointwise consistency only
- The current FleCSPH version only supports the MPI backend and does not take advantage of all FleCSI features
- Spack-based module-support installation instructions are truncated in the article ('Use the following command to install _module_ support for spack an')

## Evidence

- [methods] Copy and edit input parameter file: "cp ../data/sodtube_t1_n1000.par sodtube.par"
- [readme] Parameter files location: "Sample parameter files and the intial data can be found in the `data` subdirectory."
- [readme] Available executables via Spack: [evidence span withheld — anchor status: not_found]
- [intro] Provided example problems: "We provide several examples of physics problems in 1D, 2D and 3D: - Sod shock tubes in 1D/2D/3D;"
