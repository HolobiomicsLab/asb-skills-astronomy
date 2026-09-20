---
name: oscillation-diagnostics-extraction-from-h5part
description: Use when reproducing a FleCSPH oscillating star test across particle
  counts, or when validating SPH self-gravity evolution by extracting oscillation
  diagnostics from H5part simulation output for a diagnostics-versus-particle-number
  comparison.
when_to_use_negative:
- The test problem is a Sod shock tube, Noh shock, Sedov blast wave, or airfoil flow
  example — those use different generators and drivers and have their own diagnostics
- The goal is exact N-body gravity benchmarks where quadratic-complexity force sums
  are required rather than FMM-based self-gravity
- The simulation framework is not FleCSPH or the output is not in H5part format
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3315
tools:
- name: FleCSPH
  role: Provides the oscillating star parameter files, initial-data generators, and
    the newtonian evolution driver whose H5part output is analyzed
  repo: github.com/laristra/flecsph
- name: Spack
  role: Installs and loads FleCSPH so generators and drivers are available as executables
  repo: github.com/spack/spack
- name: FleCSI
  role: Underlying parallel octree/FMM framework exercised by the FleCSPH runs being
    validated
  repo: github.com/laristra/flecsi
- name: H5part
  role: I/O format of the simulation output from which oscillation diagnostics are
    extracted
- name: MPI
  role: Backend used by the current FleCSPH version when running the newtonian driver
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.softx.2020.100602
    title: 'FleCSPH: The next generation FleCSIble parallel computational infrastructure
      for smoothed particle hydrodynamics'
  source_content_hash: sha256:3b627968d28d91c5c12731f67da9ca3920760632c8b8fb4e9e382d8ba36d400c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FleCSI
  - FleCSPH
  - H5part
  - MPI
  - Spack
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# oscillation-diagnostics-extraction-from-h5part

## Summary

Extracts velocity and density oscillation diagnostics from FleCSPH oscillating-star simulations run at multiple particle counts, compiling them into a per-count CSV to demonstrate validation/convergence behavior. Grounded in the FleCSPH SoftwareX paper and laristra/flecsph README.

## When to use

Use when reproducing a FleCSPH oscillating star test across particle counts, or when validating SPH self-gravity evolution by extracting oscillation diagnostics from H5part simulation output for a diagnostics-versus-particle-number comparison.

## When NOT to use

- The test problem is a Sod shock tube, Noh shock, Sedov blast wave, or airfoil flow example — those use different generators and drivers and have their own diagnostics
- The goal is exact N-body gravity benchmarks where quadratic-complexity force sums are required rather than FMM-based self-gravity
- The simulation framework is not FleCSPH or the output is not in H5part format

## Inputs

- FleCSPH oscillating star .par parameter file from the 'data' subdirectory (edited per target particle count)
- Initial particle distribution produced by an oscillating star generator from app/id_generators
- Simulation output from the 'newtonian' evolution driver written in H5part format

## Outputs

- oscillating_star_diagnostics.csv with one row per tested particle count containing velocity and density oscillation diagnostics
- Optional diagnostic-versus-particle-number plot

## How to apply

Install FleCSPH via Spack ('spack install flecsph' then 'spack load flecsph') so the initial-data generators and the newtonian_3d driver are on PATH. Clone laristra/flecsph recursively and locate the oscillating star .par parameter files and initial data in the 'data' subdirectory. For each particle count used in the paper, copy the oscillating star .par file, edit the target particle number, run the matching initial-data generator from app/id_generators, then evolve with the 'newtonian' driver (3D hydro with self-gravity via FMM). Extract velocity and density oscillation diagnostics from the H5part-formatted output of each run. Compile one row per particle count into oscillating_star_diagnostics.csv and check that the oscillation behavior is consistent with the paper's reported convergence trends.

## Related tools

- **FleCSPH** (Provides the oscillating star parameter files, initial-data generators, and the newtonian evolution driver whose H5part output is analyzed) — github.com/laristra/flecsph
- **Spack** (Installs and loads FleCSPH so generators and drivers are available as executables) — github.com/spack/spack
- **FleCSI** (Underlying parallel octree/FMM framework exercised by the FleCSPH runs being validated) — github.com/laristra/flecsi
- **H5part** (I/O format of the simulation output from which oscillation diagnostics are extracted)
- **MPI** (Backend used by the current FleCSPH version when running the newtonian driver)

## Examples

```
spack load flecsph && git clone --recursive git@github.com:laristra/flecsph.git && cp flecsph/data/oscillating_star.par my_star.par && <newtonian_3d> my_star.par
```

## Evaluation signals

- oscillating_star_diagnostics.csv exists and contains one row per tested particle count
- Extracted velocity and density oscillation diagnostics show behavior consistent with the paper's reported convergence trends
- Each simulation ran the 'newtonian' driver (3D hydro with self-gravity via FMM) on initial data generated by the matching oscillating star generator
- The edited .par file's target particle count matches the count intended for that run
- Output files parse as valid H5part before diagnostics are extracted

## Limitations

- The current version of FleCSPH only supports the MPI backend and does not use all FleCSI features
- FMM timing and accuracy depend on the MAC angle, which can affect the self-gravity driving the star's oscillations
- FMM conserves linear momentum; angular momentum is only conserved exactly at first order of the multipole expansion
- The paper's example list (Sod, Noh, Sedov, airfoil) does not enumerate the oscillating star in the intro; parameter files must be located in the 'data' subdirectory of the repo

## Evidence

- [abstract] Data input/output uses the H5part format, compatible with modern visualization software.: "Data input/output uses the H5part format, compatible with modern visualization software."
- [intro] FleCSPH includes support for gravitational forces using the fast multipole method (FMM): "FleCSPH includes support for gravitational forces using the fast multipole method (FMM)."
- [readme] Sample parameter files and the intial data can be found in the `data` subdirectory.: "Sample parameter files and the intial data can be found in the `data` subdirectory."
- [methods] git clone --recursive git@github.com:laristra/flecsph.git: "git clone --recursive git@github.com:laristra/flecsph.git"
- [methods] spack install flecsph: "spack install flecsph"
- [readme] single and binary stars with Newtonian gravity in 3D: "single and binary stars with Newtonian gravity in 3D."
