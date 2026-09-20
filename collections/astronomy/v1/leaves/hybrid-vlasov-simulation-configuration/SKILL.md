---
name: hybrid-vlasov-simulation-configuration
description: Use when the task requires setting up and executing Vlasiator simulations
  (e.g., 1D collisionless shock runs at 1000 km or 200 km grid resolution, low-β wave
  tests, or multi-population proton/O+ configurations) and analysing .vlsv outputs.
when_to_use_negative:
- The analysis goal only requires single-fluid MHD behaviour with no kinetic (non-fluid)
  physics — cheaper MHD forerunners suffice when departures from fluid theory are
  not of interest
- A global MHD-level magnetosphere model is adequate and the immense computational
  cost of multi-dimensional ion-scale Vlasov grids (requiring tens of thousands of
  cores) cannot be justified
- Only spacecraft observation processing is needed rather than forward simulation
  (use in situ mission data such as Cluster instead of a Vlasov solver)
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: Vlasiator
  role: Hybrid-Vlasov solver used to run the 1D shock, wave-dispersion, and multi-population
    simulations and to write .vlsv outputs
  repo: https://github.com/fmihpc/vlasiator
- name: Analysator
  role: Python toolset for reading and analysing Vlasiator VLSV output files, including
    VDF plotting and moment extraction
  repo: https://github.com/fmihpc/analysator
- name: WHAMP
  role: Reference solver for linear hot-plasma dispersion solutions used to validate
    simulated dispersion curves
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_002
  source_papers:
  - doi: 10.1007/s41115-018-0003-2
    title: Vlasov methods in space physics and astrophysics
  source_content_hash: sha256:ce39c53f5705ee153c225bf61baedb6d220acb236c3fff25e3f2a0c178e2ee3f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fmihpc/vlasiator
  tools:
  - Analysator
  - Vlasiator
  - WHAMP
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# hybrid-vlasov-simulation-configuration

## Summary

Configure, build, and run global or 1D hybrid-Vlasov plasma simulations with Vlasiator, in which ions are represented as noiseless velocity distribution functions and electrons as an MHD fluid, for benchmarks such as collisionless shocks, wave dispersion, and multi-ion-population runs.

## When to use

Use when the task requires setting up and executing Vlasiator simulations (e.g., 1D collisionless shock runs at 1000 km or 200 km grid resolution, low-β wave tests, or multi-population proton/O+ configurations) and analysing .vlsv outputs.

## When NOT to use

- The analysis goal only requires single-fluid MHD behaviour with no kinetic (non-fluid) physics — cheaper MHD forerunners suffice when departures from fluid theory are not of interest
- A global MHD-level magnetosphere model is adequate and the immense computational cost of multi-dimensional ion-scale Vlasov grids (requiring tens of thousands of cores) cannot be justified
- Only spacecraft observation processing is needed rather than forward simulation (use in situ mission data such as Cluster instead of a Vlasov solver)

## Inputs

- Vlasiator run configuration files (1D shock benchmark with terrestrial bow-shock-like plasma conditions per Pfau-Kempf et al. 2018; 1D low-β wave-test setups at parallel, perpendicular, oblique angles; two-population proton/O+ setup)
- Built Vlasiator solver from github.com/fmihpc/vlasiator (cloned with --recurse-submodules)
- WHAMP linear hot-plasma dispersion reference solutions (Rönnmark 1982) for matching plasma parameters and angles

## Outputs

- Vlasiator output .vlsv data files containing density and bulk-velocity fields per ion population
- shock_comparison_table.csv with magnetic compression ratio and non-Maxwellianity metrics at 1000 km and 200 km resolution
- dispersion_curves.csv (ω vs k per propagation angle) plus a maximum relative frequency error score vs WHAMP

## How to apply

Build Vlasiator by cloning with submodules (`git clone --recurse-submodules https://github.com/fmihpc/vlasiator` followed by `git submodule update --init --recursive`) and follow the Installing-Vlasiator wiki page for dependencies and compilation. Configure the run following the Pfau-Kempf et al. (2018) benchmark setups: for a 1D collisionless shock, set terrestrial bow-shock-like plasma conditions and run at 1000 km spatial resolution, then repeat identically at 200 km to assess resolution-dependent kinetic detail; for wave verification, launch 1D low-β runs at parallel, perpendicular, and oblique propagation angles and extract ω(k) dispersion via spatiotemporal Fourier decomposition, comparing against WHAMP (Rönnmark 1982) linear solutions; for multi-species, add a second ion population (e.g., O+) on a sparse velocity-space grid, localise its density to the near-Earth ionospheric region, and enable per-species density and bulk-velocity outputs. Analyse downstream shock regions for magnetic compression ratio and non-Maxwellianity of the ion distribution, or read .vlsv outputs with Analysator (`import analysator as pt`). The rationale is that modelling ions as velocity distribution functions yields numerically noiseless results that capture non-MHD, multi-temperature processes that fluid (MHD) models cannot self-consistently describe.

## Related tools

- **Vlasiator** (Hybrid-Vlasov solver used to run the 1D shock, wave-dispersion, and multi-population simulations and to write .vlsv outputs) — https://github.com/fmihpc/vlasiator
- **Analysator** (Python toolset for reading and analysing Vlasiator VLSV output files, including VDF plotting and moment extraction) — https://github.com/fmihpc/analysator
- **WHAMP** (Reference solver for linear hot-plasma dispersion solutions used to validate simulated dispersion curves)

## Examples

```
git clone --recurse-submodules https://github.com/fmihpc/vlasiator && cd vlasiator && git submodule update --init --recursive
```

## Evaluation signals

- Shock benchmark: magnetic compression ratio in the downstream shock region falls near the ~4x intensification reported for the terrestrial bow shock, and required fields are present in shock_comparison_table.csv
- Resolution check: the 200 km run shows improved physical detail over the 1000 km run, and both runs depart from fluid theory consistently with a kinetic description
- Wave benchmark: Vlasiator dispersion curves agree excellently with WHAMP solutions for parallel, perpendicular, and oblique propagation, with deviations only at high frequencies and wave numbers
- Multi-population check: output .vlsv file contains nonzero O+ density and bulk-velocity fields, with O+ density localised to the near-Earth (ionospheric) region
- Build/process check: Vlasiator compiles after submodule initialisation and the analysis script runs to completion; expert review confirms kinetic departures from fluid theory

## Limitations

- Vlasiator's computational challenges are immense due to the multi-dimensional approach at ion scales, requiring advanced HPC techniques and massively parallel computations on tens of thousands of cores
- In wave-dispersion verification, noticeable differences from WHAMP linear theory appear at high frequencies and wave numbers
- When cloning, dependent header libraries are moving to git submodules, so omitting `--recurse-submodules` / `git submodule update --init --recursive` can yield an incomplete build

## Evidence

- [readme] In Vlasiator, ions are represented as velocity distribution functions, while electrons are magnetohydrodynamic fluid, enabling a self-consistent global plasma simulation: [evidence span withheld — anchor status: external_doc]
- [other] Even at coarse resolution the Vlasov results depart from fluid theory: [evidence span withheld — anchor status: not_found]
- [other] Excellent agreement between Vlasiator and WHAMP dispersion curves: [evidence span withheld — anchor status: not_found]
- [methods] Vlasiator installation wiki instructions: "https://github.com/fmihpc/vlasiator/wiki/Installing-Vlasiator"
- [readme] Submodule cloning requirement: [evidence span withheld — anchor status: external_doc]
