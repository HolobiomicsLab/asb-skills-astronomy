---
name: sph-parameter-file-configuration
description: Use when setting up a FleCSPH simulation run (e.g. Sod tube, Sedov, oscillating
  star) and you need to select, copy, and edit a .par parameter file to define particle
  count and match it to the right generator and driver executable.
when_to_use_negative:
- The target problem is not among FleCSPH's provided examples (Sod shock tubes, Noh
  shock test, Sedov blast waves, airfoil flow, spherical implosion, single/binary
  stars) and no template .par file exists
- You need a runtime backend other than MPI, since the current FleCSPH version is
  limited to MPI support and does not use all FleCSI features
- The goal is a non-SPH mesh-based method, since FleCSPH implements a smoothed-particle
  hydrodynamics (SPH) Lagrangian solver specifically
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3365
tools:
- name: FleCSPH
  role: SPH simulation tool whose .par parameter files, id_generators, and drivers
    (hydro_1-2-3d, newtonian_3d) are configured by this skill
  repo: https://github.com/laristra/flecsph
- name: Spack
  role: Installs FleCSPH ('spack install flecsph') and exposes the generator and driver
    executables in the environment via 'spack load flecsph'
  repo: https://github.com/spack/spack
- name: H5part
  role: Output data format written by the evolution drivers for visualization
- name: FleCSI
  role: Compile-time configurable framework providing the parallel octree data structures
    underlying FleCSPH runs configured via parameter files
  repo: https://github.com/laristra/flecsi
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.softx.2020.100602
    title: 'FleCSPH: The next generation FleCSIble parallel computational infrastructure
      for smoothed particle hydrodynamics'
  source_content_hash: sha256:31d25d38e1c628143b4050bf1d93ed558f1aeb44c762e67a09b4fd963ff3b68a
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
  - http://edamontology.org/topic_3365
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# sph-parameter-file-configuration

## Summary

Configure FleCSPH SPH simulations by copying and editing pre-configured .par parameter files from the repository's data subdirectory, setting target particle counts and pairing them with the correct initial-data generator and evolution driver (e.g. hydro_1-2-3d or newtonian_3d).

## When to use

Use when setting up a FleCSPH simulation run (e.g. Sod tube, Sedov, oscillating star) and you need to select, copy, and edit a .par parameter file to define particle count and match it to the right generator and driver executable.

## When NOT to use

- The target problem is not among FleCSPH's provided examples (Sod shock tubes, Noh shock test, Sedov blast waves, airfoil flow, spherical implosion, single/binary stars) and no template .par file exists
- You need a runtime backend other than MPI, since the current FleCSPH version is limited to MPI support and does not use all FleCSI features
- The goal is a non-SPH mesh-based method, since FleCSPH implements a smoothed-particle hydrodynamics (SPH) Lagrangian solver specifically

## Inputs

- Pre-configured FleCSPH .par parameter file from the repository's data subdirectory (e.g. data/sodtube_t1_n1000.par, oscillating star .par files)
- FleCSPH installation with Spack-provided generators and drivers (sodtube_{1-2-3}d_generator, sedov_{2-3}d_generator, hydro_{1-2-3}d, newtonian_3d)
- Desired target particle count for the test problem

## Outputs

- Edited .par parameter file configured for the chosen problem and particle count
- Initial particle distribution produced by the matching id_generator
- H5part-format simulation output from the hydro or newtonian evolution driver

## How to apply

Clone the FleCSPH repository recursively and locate pre-configured parameter files and initial data in the 'data' subdirectory. Copy the example .par file closest to your physics problem (e.g. 'cp ../data/sodtube_t1_n1000.par sodtube.par') and edit it to set the target number of particles, as done for the oscillating star cases across particle counts in the paper. Pair the parameter file with the matching initial-data generator (e.g. sodtube_{1-2-3}d_generator, sedov_{2-3}d_generator) built under app/id_generators, then run the appropriate driver — hydro_{1-2-3}d for pure hydrodynamics or newtonian_3d for 3D hydro with self-gravity via FMM. Ensure executables are exposed first via 'spack install flecsph' followed by 'spack load flecsph'. Simulation output is written in H5part format, compatible with modern visualization software.

## Related tools

- **FleCSPH** (SPH simulation tool whose .par parameter files, id_generators, and drivers (hydro_1-2-3d, newtonian_3d) are configured by this skill) — https://github.com/laristra/flecsph
- **Spack** (Installs FleCSPH ('spack install flecsph') and exposes the generator and driver executables in the environment via 'spack load flecsph') — https://github.com/spack/spack
- **H5part** (Output data format written by the evolution drivers for visualization)
- **FleCSI** (Compile-time configurable framework providing the parallel octree data structures underlying FleCSPH runs configured via parameter files) — https://github.com/laristra/flecsi

## Examples

```
spack install flecsph && spack load flecsph && cp ../data/sodtube_t1_n1000.par sodtube.par
```

## Evaluation signals

- The edited .par file runs to completion under the matching driver (hydro_{1-2-3}d or newtonian_3d) without parameter-parsing errors
- The generated initial-data file's particle count matches the target particle count set in the parameter file
- Output is produced in H5part format as expected by the pipeline
- For the oscillating star, the velocity and density oscillation diagnostics extracted from output are consistent with the paper's reported convergence trends across particle counts
- The parameter file was derived from the correct physics template in the data subdirectory (e.g. sodtube, sedov, oscillating star)

## Limitations

- FleCSPH currently only supports the MPI backend and does not take advantage of all FleCSI features
- Repository documentation is truncated in places (e.g. the Spack module-support installation instruction is cut off mid-sentence), so some setup steps may need to be inferred
- No changelog/discussion content was available, so parameter-file option semantics beyond the paper's examples may require reading source

## Evidence

- [readme] Sample parameter files and the intial data can be found in the `data` subdirectory.: "Sample parameter files and the intial data can be found in the `data` subdirectory."
- [methods] Copy and edit input parameter file from methods section: "cp ../data/sodtube_t1_n1000.par sodtube.par"
- [readme] You will then have access to the generators and the drivers: "You will then have access to the generators and the drivers: - sodtube\_{1-2-3}d\_generator, sedov\_{2-3}d\_generator... - hydro\_{1-2-3}d, newtonian\_3d..."
- [abstract] Data input/output uses the H5part format, compatible with modern visualization software.: "Data input/output uses the H5part format, compatible with modern visualization software."
- [intro] FleCSPH can now be installed as a Spack package: "FleCSPH can now be installed as a Spack package"
