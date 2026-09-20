---
name: bow-shock-plasma-parameterization
description: Use when you must test whether a kinetic plasma model reproduces non-fluid
  shock physics at bow-shock-scale resolutions, e.g. comparing magnetic compression
  ratios and downstream ion distribution non-Maxwellianity across grid resolutions.
when_to_use_negative:
- The research question only requires single-fluid MHD behaviour; Vlasiator's hybrid-Vlasov
  ion-distribution approach is computationally far more expensive than MHD forerunners.
- The system is not a collisionless shock / near-Earth plasma problem — the article's
  scope is the Earth's magnetosphere and near-Earth environment, and the hope of extending
  to astrophysical applications is explicitly future work.
- Only 3D global magnetosphere dynamics are needed rather than 1D shock-resolved kinetic
  physics; the immense computational challenges of multi-dimensional ion-scale runs
  may not be justified.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: Vlasiator
  role: Global hybrid-Vlasov solver used to run the 1D collisionless shock benchmark
    at 1000 km and 200 km resolutions and produce VLSV output files
  repo: https://github.com/fmihpc/vlasiator
- name: analysator
  role: Python toolset to read and analyse the VLSV output files, e.g. extracting
    downstream magnetic compression ratio and plotting ion velocity distribution functions
    (plot_vdf)
  repo: https://github.com/fmihpc/analysator
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s41115-018-0003-2
    title: Vlasov methods in space physics and astrophysics
  source_content_hash: sha256:24d0b8bc7eadfa72e77dbec71726ae61589afd67f05eb4b9c6dbc64fb5add016
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fmihpc/vlasiator
  tools:
  - Vlasiator
  - analysator
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# bow-shock-plasma-parameterization

## Summary

Parameterize a 1D collisionless shock using the Vlasiator hybrid-Vlasov code at terrestrial bow-shock-like plasma conditions, quantifying kinetic (non-fluid) behaviour via downstream magnetic compression ratio and non-Maxwellianity of the ion distribution at coarse (1000 km) and refined (200 km) resolutions.

## When to use

Use when you must test whether a kinetic plasma model reproduces non-fluid shock physics at bow-shock-scale resolutions, e.g. comparing magnetic compression ratios and downstream ion distribution non-Maxwellianity across grid resolutions.

## When NOT to use

- The research question only requires single-fluid MHD behaviour; Vlasiator's hybrid-Vlasov ion-distribution approach is computationally far more expensive than MHD forerunners.
- The system is not a collisionless shock / near-Earth plasma problem — the article's scope is the Earth's magnetosphere and near-Earth environment, and the hope of extending to astrophysical applications is explicitly future work.
- Only 3D global magnetosphere dynamics are needed rather than 1D shock-resolved kinetic physics; the immense computational challenges of multi-dimensional ion-scale runs may not be justified.

## Inputs

- Vlasiator solver built from github.com/fmihpc/vlasiator (cloned with --recurse-submodules)
- 1D shock benchmark configuration with terrestrial bow-shock-like plasma conditions (Pfau-Kempf et al. 2018 setup)
- Two grid resolutions: 1000 km (coarse) and 200 km (refined)

## Outputs

- Vlasiator VLSV output data files from both resolution runs
- shock_comparison_table.csv with per-resolution compression ratio and non-Maxwellianity metrics
- Analysis scripts/plots of downstream ion distribution functions

## How to apply

Compile and install Vlasiator following the repository wiki's installation instructions, cloning with submodules (`git clone --recurse-submodules https://github.com/fmihpc/vlasiator`). Configure the 1D collisionless shock benchmark with terrestrial bow-shock-like plasma conditions following the Pfau-Kempf et al. (2018) setup, and run the solver twice: once at 1000 km grid resolution and once at the refined 200 km resolution, writing VLSV output files. Analyse the downstream shock region in both runs (e.g. with the fmihpc/analysator Python tools that read VLSV files), extracting the magnetic compression ratio and computing a non-Maxwellianity metric for the downstream ion distribution. The rationale is that Vlasiator models ions as velocity distribution functions (electrons as MHD fluid), so it captures multi-temperature, non-MHD physics that single-fluid MHD forerunners miss; accept the run when compression ratios fall near the ~4x terrestrial bow-shock intensification and the 200 km run shows improved physical detail consistent with a kinetic description.

## Related tools

- **Vlasiator** (Global hybrid-Vlasov solver used to run the 1D collisionless shock benchmark at 1000 km and 200 km resolutions and produce VLSV output files) — https://github.com/fmihpc/vlasiator
- **analysator** (Python toolset to read and analyse the VLSV output files, e.g. extracting downstream magnetic compression ratio and plotting ion velocity distribution functions (plot_vdf)) — https://github.com/fmihpc/analysator

## Examples

```
git clone --recurse-submodules https://github.com/fmihpc/vlasiator && pip install --editable ./analysator; python -c "import analysator as pt; pt.plot.plot_vdf?"
```

## Evaluation signals

- Magnetic compression ratio in the downstream region falls in the expected range near the ~4x intensification reported for the terrestrial bow shock
- The 200 km run shows clearly improved physical detail over the 1000 km run while both depart from fluid (MHD) theory
- shock_comparison_table.csv contains the required fields (resolution, compression ratio, non-Maxwellianity) with no missing values
- The analysis script runs to completion on the VLSV outputs from both resolutions
- Expert review confirms departures from fluid theory are consistent with a kinetic description of the downstream ion distribution

## Limitations

- Vlasiator's multi-dimensional ion-scale approach poses immense computational challenges, requiring massively parallel HPC on tens of thousands of cores; 1D coarse-resolution runs are a tractable subset
- Electrons are treated as an MHD fluid, so purely electron-scale kinetic physics is not captured
- Results are validated only against terrestrial bow-shock-like conditions and the Pfau-Kempf et al. (2018) setup; extrapolation to other shock parameter regimes is untested here

## Evidence

- [readme] In Vlasiator, ions are represented as velocity distribution functions, while electrons are magnetohydrodynamic fluid: [evidence span withheld — anchor status: external_doc]
- [abstract] Vlasiator is the world's first global hybrid-Vlasov simulation: "A particular application of the Vlasov system is Vlasiator, the world’s first global hybrid-Vlasov simulation for the Earth’s magnetic domain, the magnetosphere."
- [readme] Analysator reads VLSV output files: [evidence span withheld — anchor status: not_found]
- [methods] Installing Vlasiator from the wiki: [evidence span withheld — anchor status: not_found]
- [readme] Submodule cloning requirement: [evidence span withheld — anchor status: external_doc]
