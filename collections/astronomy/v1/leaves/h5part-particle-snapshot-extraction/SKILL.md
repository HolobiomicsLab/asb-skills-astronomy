---
name: h5part-particle-snapshot-extraction
description: Use when FleCSPH hydro_{1-2-3}d or newtonian_3d drivers have written
  H5part particle snapshot files and you need to extract per-particle position, density,
  and velocity to compute radial, azimuthal, or shock profiles for validation.
when_to_use_negative:
- The input is a FleCSPH parameter file (.par) or build configuration rather than
  simulation output — snapshot extraction applies only to generated H5part files
- The data are gridded/mesh-based outputs from another code rather than particle snapshots
  in H5part format
- The goal is building or installing FleCSPH/FleCSI (Spack, CMake with MPI backend)
  rather than post-processing an existing run
tools:
- name: FleCSPH
  role: Simulation tool whose hydro_{1-2-3}d and newtonian_3d drivers write the H5part
    particle snapshots that this skill extracts fields from
  repo: github.com/laristra/flecsph
- name: H5part
  role: Data format of the particle snapshots; chosen by FleCSPH because it is compatible
    with modern visualization software
- name: FleCSI
  role: Underlying compile-time framework providing the parallel particle data structures
    that produce the snapshots
  repo: github.com/laristra/flecsi
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_002
  source_papers:
  - doi: 10.1016/j.softx.2020.100602
    title: 'FleCSPH: The next generation FleCSIble parallel computational infrastructure
      for smoothed particle hydrodynamics'
  source_content_hash: sha256:43d0eb93f84dc54369e139ca4e74584a79ddd687485419983d533039d9b27264
schema_version: 0.2.0
metadata:
  license_tier: open
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
yes_rate: 0.8333333333333334
n_claims: 6
---

# h5part-particle-snapshot-extraction

## Summary

Post-processes FleCSPH simulation output by reading particle snapshots written in the H5part format and extracting position, density, and velocity fields to build profiles for validation against reference solutions (Sod Riemann, Sedov, Noh).

## When to use

Use when FleCSPH hydro_{1-2-3}d or newtonian_3d drivers have written H5part particle snapshot files and you need to extract per-particle position, density, and velocity to compute radial, azimuthal, or shock profiles for validation.

## When NOT to use

- The input is a FleCSPH parameter file (.par) or build configuration rather than simulation output — snapshot extraction applies only to generated H5part files
- The data are gridded/mesh-based outputs from another code rather than particle snapshots in H5part format
- The goal is building or installing FleCSPH/FleCSI (Spack, CMake with MPI backend) rather than post-processing an existing run

## Inputs

- FleCSPH H5part particle snapshot files written by hydro_{1-2-3}d or newtonian_3d evolution drivers
- H5part initial-condition files produced by sodtube_{1-2-3}d_generator, sedov_{2-3}d_generator, or noh_{2-3}d_generator (for baseline/reference comparison)

## Outputs

- Tabulated density and/or velocity profiles at the final simulation time (e.g. noh_convergence_table.csv)
- Radial/azimuthally averaged density profiles with peak density and shock-front position (e.g. sedov_density_profile.png)
- Pointwise or L1/L2 error metrics of the SPH solution against the analytic reference solution

## How to apply

Locate the H5part snapshot files written by the FleCSPH evolution drivers (hydro_1d, hydro_2d, hydro_3d for hydro problems without gravity; newtonian_3d for self-gravitating setups); H5part is the project's I/O format and is compatible with modern visualization software, so the same files can also be inspected interactively. Extract position, density, and velocity from the final snapshot step and tabulate the profiles. Choose the validation statistic per test problem: for Sod shock tubes, compare the computed density and velocity profiles against the classical Riemann solution qualitatively (the paper reports no quantitative error threshold); for Sedov blast waves, compute azimuthally/radially averaged density profiles and record peak density and shock radius against the self-similar Sedov solution; for the Noh test, compute L1/L2 density errors against the analytic solution (post-shock plateau of 64x initial density for gamma=5/3) and repeat across particle resolutions to tabulate convergence rates. Initial-condition H5part files from the sodtube_{1-2-3}d_generator, sedov_{2-3}d_generator, and noh_{2-3}d_generator executables can be read with the same machinery for baseline comparison.

## Related tools

- **FleCSPH** (Simulation tool whose hydro_{1-2-3}d and newtonian_3d drivers write the H5part particle snapshots that this skill extracts fields from) — github.com/laristra/flecsph
- **H5part** (Data format of the particle snapshots; chosen by FleCSPH because it is compatible with modern visualization software)
- **FleCSI** (Underlying compile-time framework providing the parallel particle data structures that produce the snapshots) — github.com/laristra/flecsi

## Evaluation signals

- Extracted Sod density and velocity profiles qualitatively match the classical Riemann (Sod) solution by visual/pointwise consistency
- Sedov radial density profile shows the canonical self-similar structure: dense shell localized at the outward-propagating shock radius with a low-density interior cavity, in both 2D and 3D
- Noh L1/L2 density errors decrease monotonically with increasing resolution toward the analytic 64x post-shock density plateau (gamma=5/3)
- H5part files load successfully in modern visualization software, confirming format compliance
- Shock-front position extracted from the final snapshot is consistent with the expected self-similar propagation for the test case

## Limitations

- No quantitative error threshold is reported for the Sod shock tube comparison, so acceptance relies on visual/pointwise profile agreement rather than a numeric criterion
- FleCSPH's current version only supports the MPI backend, so snapshot production is tied to MPI-parallel runs and does not exploit all FleCSI features
- FMM timing depends strongly on the MAC angle, so the cost of producing snapshots for gravity-coupled problems can vary significantly with that parameter

## Evidence

- [abstract] H5part is FleCSPH's I/O format: "Data input/output uses the H5part format, compatible with modern visualization software."
- [intro] FleCSPH is an SPH solver for Lagrangian problems: "In particular, FleCSPH implements a smoothed-particle hydrodynamics (SPH) solver for the solution of Lagrangian problems in astrophysics and cosmology."
- [readme] Sample parameter files and initial data live in the data subdirectory: "Sample parameter files and the intial data can be found in the `data` subdirectory."
