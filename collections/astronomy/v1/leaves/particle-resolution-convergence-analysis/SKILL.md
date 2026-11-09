---
name: particle-resolution-convergence-analysis
description: Use when a FleCSPH (or similar SPH) simulation must be repeated across
  multiple particle counts to check how oscillation or hydrodynamic diagnostics converge
  with resolution.
when_to_use_negative:
- The problem of interest is one of the 1D/2D shock tests (Sod tube, Noh, Sedov) rather
  than a 3D self-gravitating star — those use the hydro_{1-2-3d} drivers and their
  own generators
- You only need a single-resolution production run with no convergence assessment
- The application requires a task-based runtime backend other than MPI — the current
  FleCSPH version is limited to the MPI backend only
tools:
- name: FleCSPH
  role: SPH simulation application providing the oscillating star example, parameter
    files, generators, and drivers
  repo: github.com/laristra/flecsph
- name: FleCSI
  role: Compile-time configurable framework supplying the parallel octree data structures
    underlying the particle methods
  repo: github.com/laristra/flecsi
- name: Spack
  role: Package manager used to install and load FleCSPH executables ('spack install
    flecsph', 'spack load flecsph')
  repo: github.com/spack/spack
- name: H5part
  role: I/O format of simulation output from which oscillation diagnostics are extracted
- name: MPI
  role: Only supported backend on which the newtonian evolution driver runs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.softx.2020.100602
    title: 'FleCSPH: The next generation FleCSIble parallel computational infrastructure
      for smoothed particle hydrodynamics'
  source_content_hash: sha256:a6a5a9daeb8e034c29396b64803be784344accfd9ab8fc0cbf212c7b2475f051
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
yes_rate: 0.6
n_claims: 10
---

# particle-resolution-convergence-analysis

## Summary

Runs the FleCSPH oscillating star test at several target particle counts, evolves each with the Newtonian-gravity SPH driver, and compiles H5part-derived oscillation diagnostics (velocity/density) into a per-resolution table to assess convergence behavior like the paper's test cases.

## When to use

Use when a FleCSPH (or similar SPH) simulation must be repeated across multiple particle counts to check how oscillation or hydrodynamic diagnostics converge with resolution.

## When NOT to use

- The problem of interest is one of the 1D/2D shock tests (Sod tube, Noh, Sedov) rather than a 3D self-gravitating star — those use the hydro_{1-2-3d} drivers and their own generators
- You only need a single-resolution production run with no convergence assessment
- The application requires a task-based runtime backend other than MPI — the current FleCSPH version is limited to the MPI backend only

## Inputs

- Oscillating star .par parameter files from the flecsph 'data' subdirectory
- Initial-data generator executables from app/id_generators
- Target particle counts (one per resolution being tested, matching the paper's counts)

## Outputs

- Generated initial particle distributions for each particle count
- H5part simulation output per resolution
- oscillating_star_diagnostics.csv with one row per tested particle count
- Optional diagnostic-versus-particle-number plot

## How to apply

Install FleCSPH via Spack ('spack install flecsph') and expose the initial-data generators and drivers with 'spack load flecsph'. Clone the repository recursively ('git clone --recursive git@github.com:laristra/flecsph.git') and locate the oscillating star parameter files and initial data in the 'data' subdirectory. For each particle count used in the paper, copy the oscillating star .par file, edit the target number of particles, and run the matching initial-data generator from app/id_generators to build the particle distribution. Evolve each setup with the 'newtonian' driver (3D SPH hydrodynamics with self-gravity computed via the FMM on FleCSI's parallel octree), then extract velocity and density oscillation diagnostics from the H5part output for each count. Rationale: the paper's validation cases were both based on the oscillating star run at different particle counts, and FMM conserves linear momentum (with angular momentum exact at first multipole order), so oscillation behavior should remain stable as resolution changes.

## Related tools

- **FleCSPH** (SPH simulation application providing the oscillating star example, parameter files, generators, and drivers) — github.com/laristra/flecsph
- **FleCSI** (Compile-time configurable framework supplying the parallel octree data structures underlying the particle methods) — github.com/laristra/flecsi
- **Spack** (Package manager used to install and load FleCSPH executables ('spack install flecsph', 'spack load flecsph')) — github.com/spack/spack
- **H5part** (I/O format of simulation output from which oscillation diagnostics are extracted)
- **MPI** (Only supported backend on which the newtonian evolution driver runs)

## Examples

```
spack install flecsph && spack load flecsph
```

## Evaluation signals

- oscillating_star_diagnostics.csv exists and contains exactly one row per tested particle count
- Oscillation diagnostics extracted from H5part output show behavior consistent with the paper's reported convergence trends
- The evolved star remains stably oscillating rather than dispersing at every resolution
- Each tested particle count had a successfully generated initial distribution from the matching app/id_generators executable
- FMM's exact linear momentum conservation (angular momentum exact at first multipole order) is preserved in the outputs

## Limitations

- The current FleCSPH version does not take advantage of all FleCSI features and is limited to only the MPI backend
- Timing of the FMM algorithm strongly depends on the MAC angle, so runtime comparisons across resolutions must fix that parameter
- The article's Spack module-support installation instruction is truncated mid-sentence, leaving part of the environment setup underspecified

## Evidence

- [abstract] H5part output format: "Data input/output uses the H5part format, compatible with modern visualization software."
- [intro] SPH scope: "FleCSPH implements a smoothed-particle hydrodynamics (SPH) solver for the solution of Lagrangian problems in astrophysics and cosmology."
- [intro] FMM gravity support: "FleCSPH includes support
for gravitational forces using the fast multipole method (FMM)."
- [methods] parameter file copy-edit workflow: "cp ../data/sodtube_t1_n1000.par sodtube.par"
- [readme] data subdirectory location: "Sample parameter files and the intial data can be found in the `data` subdirectory."
- [readme] executable access after install: "In order to use FleCSPH executables simply run: 
```{engine=sh}
spack load flecsph"
