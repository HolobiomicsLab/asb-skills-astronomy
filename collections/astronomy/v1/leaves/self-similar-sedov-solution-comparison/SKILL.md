---
name: self-similar-sedov-solution-comparison
description: 'Use when verifying or benchmarking an SPH/hydro code on a strong point-explosion
  problem: the goal is to check that a simulated blast wave reproduces the canonical
  self-similar Sedov density structure in 2D or 3D.'
when_to_use_negative:
- The target problem includes self-gravity or long-range forces — use FleCSPH's gravity-enabled
  path (e.g. newtonian_3d driver with FMM) instead of the hydro-only Sedov setup
- You need a pure 1D shock-capture check — the paper provides Sod shock tubes in 1D/2D/3D,
  whereas Sedov is only offered in 2D and 3D
- The goal is testing gravitational dynamics (single/binary stars) rather than hydrodynamic
  shock structure
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_0003
tools:
- name: FleCSPH
  role: SPH application providing the sedov_{2-3}d_generator initial-data generators
    and hydro_{2-3}d evolution drivers, plus pre-configured parameter files in data/
  repo: https://github.com/laristra/flecsph
- name: FleCSI
  role: Compile-time configurable framework providing the parallel octree data structures
    on which FleCSPH's particle methods run
  repo: https://github.com/laristra/flecsi
- name: H5part
  role: Particle snapshot format in which simulation outputs are written for profile
    extraction and visualization
- name: Spack
  role: Package manager used to install FleCSPH and its dependencies and to load the
    generator/driver executables into the environment
  repo: https://github.com/spack/spack
- name: MPI
  role: Distributed-memory backend supported by FleCSPH for running the Sedov evolution
    in parallel
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.softx.2020.100602
    title: 'FleCSPH: The next generation FleCSIble parallel computational infrastructure
      for smoothed particle hydrodynamics'
  source_content_hash: sha256:1774f9a86b4c923e5b772724ef55ce485f8ad298f08dfe0633e89ba96e3eb519
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_0003
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# Self-similar Sedov solution comparison

## Summary

Validates an SPH code's hydro solver by running the FleCSPH Sedov blast-wave cases (2D and 3D), extracting azimuthally/radially averaged density profiles from H5part snapshots, and comparing them against the analytical self-similar Sedov solution.

## When to use

Use when verifying or benchmarking an SPH/hydro code on a strong point-explosion problem: the goal is to check that a simulated blast wave reproduces the canonical self-similar Sedov density structure in 2D or 3D.

## When NOT to use

- The target problem includes self-gravity or long-range forces — use FleCSPH's gravity-enabled path (e.g. newtonian_3d driver with FMM) instead of the hydro-only Sedov setup
- You need a pure 1D shock-capture check — the paper provides Sod shock tubes in 1D/2D/3D, whereas Sedov is only offered in 2D and 3D
- The goal is testing gravitational dynamics (single/binary stars) rather than hydrodynamic shock structure

## Inputs

- FleCSPH Sedov parameter file (.par) copied from the repository's data/ subdirectory and edited for particle count and problem parameters
- sedov_2d_generator / sedov_3d_generator executables (EXT_GDIMENSION=2 or 3) to generate initial particle data
- hydro_2d / hydro_3d evolution drivers from app/drivers/
- H5part-format particle snapshots written at the final simulation time

## Outputs

- Azimuthally (2D) or radially (3D) averaged density profile at the final simulation time
- CSV summary recording peak density, shock radius, and profile values
- Overlay plot of extracted density profiles versus the self-similar Sedov solution, saved as sedov_density_profile.png

## How to apply

Copy a Sedov parameter file from FleCSPH's data/ subdirectory following the pattern of sodtube_t1_n1000.par, and edit it to set particle count and problem parameters. Build the initial condition with sedov_2d_generator or sedov_3d_generator (built under app/id_generators/ with EXT_GDIMENSION set to 2 or 3), then evolve the blast wave with the hydro_2d or hydro_3d drivers (app/drivers/, hydro evolution without gravity). Read the H5part-format particle snapshots written by the simulation — the paper notes I/O uses the H5part format compatible with modern visualization software — and compute azimuthally (2D) or radially (3D) averaged density profiles at the final simulation time. Plot the extracted profiles against the self-similar Sedov blast-wave solution, recording peak density, shock radius, and profile values in a CSV summary and saving the figure as sedov_density_profile.png. Rationale: Sedov is a pure-hydro strong-shock test, so agreement with the self-similar solution (dense shell at the outward-propagating shock front enclosing a low-density interior cavity) is a stringent check of the SPH solver's shock capture without gravitational forces.

## Related tools

- **FleCSPH** (SPH application providing the sedov_{2-3}d_generator initial-data generators and hydro_{2-3}d evolution drivers, plus pre-configured parameter files in data/) — https://github.com/laristra/flecsph
- **FleCSI** (Compile-time configurable framework providing the parallel octree data structures on which FleCSPH's particle methods run) — https://github.com/laristra/flecsi
- **H5part** (Particle snapshot format in which simulation outputs are written for profile extraction and visualization)
- **Spack** (Package manager used to install FleCSPH and its dependencies and to load the generator/driver executables into the environment) — https://github.com/spack/spack
- **MPI** (Distributed-memory backend supported by FleCSPH for running the Sedov evolution in parallel)

## Examples

```
spack load flecsph; sedov_2d_generator; hydro_2d sedov_2d.par
```

## Evaluation signals

- Final radial density profile shows the canonical self-similar Sedov structure: a dense shell localized at the outward-propagating shock radius with a low-density interior cavity
- The same structure is reproduced for both the 2D and the 3D Sedov cases
- Peak density and shock radius extracted from the H5part snapshots fall on the self-similar Sedov curve when overlaid in sedov_density_profile.png
- Simulated shock front remains spherically (2D: circularly) symmetric after azimuthal/radial averaging, with no spurious asymmetries
- The generator ran with the correct EXT_GDIMENSION (2 vs 3) matching the driver used for evolution

## Limitations

- Sedov blast waves are provided only in 2D and 3D — no 1D Sedov case exists, so 1D checks must use the Sod shock tube instead
- The hydro_2d/hydro_3d drivers evolve hydrodynamics without gravity, so the Sedov comparison cannot validate gravitational-force handling (FMM)
- The current FleCSPH version is limited to the MPI backend and does not take advantage of all FleCSI features
- Spack's module-support installation instruction is truncated in the paper, which may complicate environment setup before running the generators

## Evidence

- [readme] Sedov blast waves 2D and 3D;: "We provide several examples of physics problems in 1D, 2D and 3D:

- Sod shock tubes in 1D/2D/3D;
- Noh shock test in 2D/3D;
- Sedov blast waves 2D and 3D;"
- [abstract] Data input/output uses the H5part format, compatible with modern visualization software.: "Data input/output uses the H5part format, compatible with modern visualization software."
- [readme] generators and drivers listed in README: "You will then have access to the generators and the drivers: 
- sodtube\_{1-2-3}d\_generator, sedov\_{2-3}d\_generator..."
- [readme] parameter files location: "Sample parameter files and the intial data can be found in the `data` subdirectory."
- [intro] SPH solver scope: "FleCSPH
implements a smoothed-particle hydrodynamics (SPH) solver for the solution of
Lagrangian problems in astrophysics and cosmology."
