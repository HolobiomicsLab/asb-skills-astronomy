---
name: radial-density-profile-extraction
description: Use when H5part particle snapshots from a FleCSPH (SPH) hydro run must
  be reduced to azimuthally/radially averaged density profiles to verify Sedov blast-wave
  self-similar structure in 2D or 3D.
when_to_use_negative:
- Planar-symmetric test problems (Sod shock tube in 1D/2D/3D, Noh shock test in 2D/3D)
  where profiles are extracted along the shock direction rather than by radial/azimuthal
  averaging.
- Problems without radial symmetry, such as the bundled airfoil flow in a wind tunnel
  (2D/3D), for which azimuthal averaging has no physical meaning.
- 'Gravity-dominated problems: hydro_2d/hydro_3d are drivers for hydro evolution without
  gravity; use the newtonian_3d driver instead.'
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: FleCSPH
  role: SPH simulation tool whose sedov_{2-3}d_generator and hydro_{2-3}d executables
    produce and evolve the blast wave whose snapshots are profiled
  repo: github.com/laristra/flecsph
- name: H5part
  role: Data format of the particle snapshots that are read to compute the radial
    density profiles
- name: FleCSI
  role: Compile-time configurable framework underlying FleCSPH's parallel particle
    data structures used in the simulation stage
  repo: github.com/laristra/flecsi
- name: Spack
  role: Package manager used to install FleCSPH and load its generator/driver executables
    into the environment
  repo: github.com/spack/spack
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.softx.2020.100602
    title: 'FleCSPH: The next generation FleCSIble parallel computational infrastructure
      for smoothed particle hydrodynamics'
  source_content_hash: sha256:670a9a808e7cd01db31713020b208b4bfaa6d3e2727927645478c1fe27f01eb2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FleCSI
  - FleCSPH
  - H5part
  - MPI
  - Spack
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# radial-density-profile-extraction

## Summary

Extracts azimuthally/radially averaged density profiles from FleCSPH SPH particle snapshots (H5part format) of Sedov blast-wave simulations in 2D and 3D, and validates them against the self-similar Sedov solution by recording peak density, shock radius, and profile values.

## When to use

Use when H5part particle snapshots from a FleCSPH (SPH) hydro run must be reduced to azimuthally/radially averaged density profiles to verify Sedov blast-wave self-similar structure in 2D or 3D.

## When NOT to use

- Planar-symmetric test problems (Sod shock tube in 1D/2D/3D, Noh shock test in 2D/3D) where profiles are extracted along the shock direction rather than by radial/azimuthal averaging.
- Problems without radial symmetry, such as the bundled airfoil flow in a wind tunnel (2D/3D), for which azimuthal averaging has no physical meaning.
- Gravity-dominated problems: hydro_2d/hydro_3d are drivers for hydro evolution without gravity; use the newtonian_3d driver instead.

## Inputs

- H5part-format particle snapshots written by FleCSPH hydro_2d / hydro_3d simulations
- FleCSPH parameter file (.par) copied from the data/ subdirectory (pattern of sodtube_t1_n1000.par) and edited for the Sedov 2D/3D case
- Initial particle data produced by sedov_2d_generator / sedov_3d_generator (EXT_GDIMENSION=2/3)

## Outputs

- Azimuthally/radially averaged density profiles at the final simulation time (2D and 3D)
- CSV summary recording peak density, shock radius, and profile values
- Figure sedov_density_profile.png overlaying extracted density profiles on the self-similar Sedov solution

## How to apply

First produce the test data: copy a parameter file from the repository's data/ subdirectory following the sodtube_t1_n1000.par pattern, edit it for the Sedov case (particle number and problem parameters), run sedov_2d_generator or sedov_3d_generator (built under app/id_generators/ with EXT_GDIMENSION set to 2 or 3) to create initial particle data, then evolve with the hydro_2d or hydro_3d driver (app/drivers/, hydro evolution without gravity). Read the H5part-format particle snapshots written by the simulation at the final time and bin particles azimuthally (2D) or radially (2D/3D) to obtain averaged density profiles. Plot the extracted 2D and 3D density profiles against the analytic self-similar Sedov blast-wave solution, recording peak density, shock radius, and profile values in a CSV summary and saving the figure as sedov_density_profile.png. Accept the run only if the final radial profile exhibits the canonical self-similar Sedov structure: a dense shell localized at the outward-propagating shock radius surrounding a low-density interior cavity, for both the 2D and 3D cases.

## Related tools

- **FleCSPH** (SPH simulation tool whose sedov_{2-3}d_generator and hydro_{2-3}d executables produce and evolve the blast wave whose snapshots are profiled) — github.com/laristra/flecsph
- **H5part** (Data format of the particle snapshots that are read to compute the radial density profiles)
- **FleCSI** (Compile-time configurable framework underlying FleCSPH's parallel particle data structures used in the simulation stage) — github.com/laristra/flecsi
- **Spack** (Package manager used to install FleCSPH and load its generator/driver executables into the environment) — github.com/spack/spack

## Examples

```
spack install flecsph; spack load flecsph; sedov_2d_generator sedov_2d.par; hydro_2d sedov_2d.par
```

## Evaluation signals

- Final radial density profile shows the canonical self-similar Sedov structure: a dense shell localized at the outward-propagating shock radius with a low-density interior cavity, in both the 2D and 3D cases.
- Extracted 2D and 3D density profiles plotted against the self-similar Sedov solution agree at the shock position and shell amplitude.
- CSV summary contains finite, physically plausible peak-density and shock-radius values recorded at the final simulation time.
- H5part snapshots are non-empty and readable at the final simulation time before binning.
- The figure sedov_density_profile.png exists and shows both simulated profiles overlaid with the analytic solution.

## Limitations

- The current FleCSPH version only supports the MPI backend and does not take advantage of all FleCSI features.
- Acceptance is structural (self-similar Sedov shell/cavity morphology), which presumes the idealized Sedov setup (point energy deposition in a uniform medium); deviations invalidate the comparison.
- The article's Spack module-support installation instruction is truncated mid-sentence ('install _module_ support for spack an'), leaving part of environment setup undocumented.

## Evidence

- [abstract] Data input/output uses the H5part format, compatible with modern visualization software.: "Data input/output uses the H5part format, compatible with modern visualization software."
- [intro] Sedov example problems in 2D and 3D are bundled with FleCSPH: "We provide several examples of physics problems in 1D, 2D and 3D:"
- [methods] Parameter files are copied from the data/ subdirectory and edited: "cp ../data/sodtube_t1_n1000.par sodtube.par"
- [readme] Sample parameter files and initial data live in the data subdirectory of the repository: "Sample parameter files and the intial data can be found in the `data` subdirectory."
- [readme] FleCSPH is a multi-physics compact application exercising FleCSI parallel data structures for tree-based particle methods: "FleCSPH is a multi-physics compact application that exercises FleCSI parallel data structures for tree-based particle methods."
