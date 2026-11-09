---
name: shock-compression-ratio-analysis
description: Use when assessing whether a collisionless-shock simulation departs from
  fluid (MHD) theory at resolutions comparable to the terrestrial bow shock (~1000
  km), by comparing compression ratios near ~4x bow-shock intensification across grid
  resolutions.
when_to_use_negative:
- The target is a multi-temperature observational study from spacecraft constellations
  (e.g. Cluster) rather than a simulation output in VLSV format
- The problem requires global 3D magnetospheric dynamics rather than a 1D shock benchmark;
  the computational cost and setup differ fundamentally
- A single-fluid MHD model suffices for the physics of interest — this skill specifically
  targets non-MHD, ion-kinetic effects that fluid forerunners miss
tools:
- name: Vlasiator
  role: Global hybrid-Vlasov solver used to run the 1D collisionless shock benchmark
    at 1000 km and 200 km resolutions and write VLSV output files
  repo: https://github.com/fmihpc/vlasiator
- name: Analysator
  role: Python tools used to read and analyse Vlasiator VLSV output files, extracting
    downstream shock quantities such as compression ratio and ion distribution non-Maxwellianity
  repo: https://github.com/fmihpc/analysator.git
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s41115-018-0003-2
    title: Vlasov methods in space physics and astrophysics
  source_content_hash: sha256:bc4daf4c30791dc5ea94f6add574f1a95a3257c3ba600fbfe20ba29e5c1e5bdb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fmihpc/vlasiator
  tools:
  - Analysator
  - Vlasiator
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# shock-compression-ratio-analysis

## Summary

A two-resolution hybrid-Vlasov benchmark skill that quantifies whether a 1D collisionless shock simulation reproduces kinetic (non-fluid) physics at coarse grid spacing, by extracting the downstream magnetic compression ratio and a non-Maxwellianity metric for the ion distribution from Vlasiator runs at 1000 km and 200 km resolution.

## When to use

Use when assessing whether a collisionless-shock simulation departs from fluid (MHD) theory at resolutions comparable to the terrestrial bow shock (~1000 km), by comparing compression ratios near ~4x bow-shock intensification across grid resolutions.

## When NOT to use

- The target is a multi-temperature observational study from spacecraft constellations (e.g. Cluster) rather than a simulation output in VLSV format
- The problem requires global 3D magnetospheric dynamics rather than a 1D shock benchmark; the computational cost and setup differ fundamentally
- A single-fluid MHD model suffices for the physics of interest — this skill specifically targets non-MHD, ion-kinetic effects that fluid forerunners miss

## Inputs

- Vlasiator solver installed from github.com/fmihpc/vlasiator (cloned with --recurse-submodules)
- 1D collisionless-shock benchmark configuration with terrestrial bow-shock-like plasma conditions (Pfau-Kempf et al. 2018 setup)
- VLSV output data files from the 1000 km resolution shock run
- VLSV output data files from the 200 km resolution shock run

## Outputs

- shock_comparison_table.csv containing magnetic compression ratio and non-Maxwellianity of the downstream ion distribution for each resolution (1000 km and 200 km)
- Analysis-script output confirming kinetic departures from fluid theory and improved physical detail at 200 km

## How to apply

Compile and install Vlasiator from its repository (cloning with --recurse-submodules to initialise dependent header libraries), then configure a 1D collisionless shock benchmark with terrestrial bow-shock-like plasma conditions following the Pfau-Kempf et al. (2018) setup. Run the shock on the solver at the coarse 1000 km grid resolution and write the VLSV output data files, then re-run the identical setup at the refined 200 km resolution. In the Vlasiator model, ions are represented as velocity distribution functions (yielding numerically noiseless, non-MHD ion-scale physics) while electrons are a magnetohydrodynamic fluid, so analyse the downstream shock region of both runs with Analysator's Python tools, extracting the magnetic compression ratio and computing a non-Maxwellianity metric for the downstream ion distribution. Assemble the two-resolution results into shock_comparison_table.csv; the rationale is that compression near ~4x plus non-Maxwellian downstream distributions indicate a kinetic, rather than fluid-like, description that improves in physical detail as resolution is refined.

## Related tools

- **Vlasiator** (Global hybrid-Vlasov solver used to run the 1D collisionless shock benchmark at 1000 km and 200 km resolutions and write VLSV output files) — https://github.com/fmihpc/vlasiator
- **Analysator** (Python tools used to read and analyse Vlasiator VLSV output files, extracting downstream shock quantities such as compression ratio and ion distribution non-Maxwellianity) — https://github.com/fmihpc/analysator.git

## Examples

```
git clone --recurse-submodules https://github.com/fmihpc/vlasiator; pip install --editable ./analysator; python analysis/1D_shock_downstream.py --runs 1000km,200km --out shock_comparison_table.csv
```

## Evaluation signals

- Magnetic compression ratio values in shock_comparison_table.csv fall in the expected range near the ~4x intensification reported for the terrestrial bow shock
- All required fields are present in shock_comparison_table.csv for both the 1000 km and 200 km runs
- The analysis script runs to completion without errors on the VLSV outputs
- Expert review confirms downstream departures from fluid theory are consistent with a kinetic description
- The 200 km run shows measurably improved physical detail (sharper downstream structure, more informative distribution function) relative to the 1000 km run

## Limitations

- Computational challenges are immense: the multi-dimensional ion-scale approach requires advanced HPC techniques and massively parallel runs on tens of thousands of cores
- Results are validated only against terrestrial bow-shock-like conditions at 1D coarse resolution; extrapolation to other regimes is untested here
- Vlasiator depends on git submodules and header libraries — cloning without --recurse-submodules will produce an incomplete build
- Improvements in physical detail are resolution-dependent; at 1000 km the kinetic signal is detectable but only the 200 km run provides dramatically improved detail

## Evidence

- [readme] Ions as distribution functions in Vlasiator: [evidence span withheld — anchor status: external_doc]
- [abstract] Vlasiator as prime Vlasov application: "A particular application of the Vlasov system is Vlasiator, the world’s first global hybrid-Vlasov simulation for the Earth’s magnetic domain, the magnetosphere."
- [abstract] Validation scope of the review: "A short review of verification, validation and physical results is included"
- [readme] Analysator reads VLSV outputs: [evidence span withheld — anchor status: not_found]
