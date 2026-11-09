---
name: collisionless-shock-initialization
description: Use when you need to reproduce a 1D collisionless-shock discretisation
  study with Vlasiator, e.g. to check whether a hybrid-Vlasov model departs from fluid
  (MHD) theory at coarse resolutions like ~1000 km.
when_to_use_negative:
- Input plasma is well described by single-fluid MHD and you only need the cheaper
  MHD-like answer; the point of this skill is to resolve departures from fluid theory,
  not to replicate MHD.
- The system of interest requires full global magnetospheric geometry (2D/3D magnetosphere
  coupling) rather than the 1D shock benchmark; this skill covers only the 1D normal-incidence-style
  shock setup.
- You need self-consistent electron kinetics — in Vlasiator electrons are treated
  as an MHD fluid, so purely electron-scale kinetic questions are out of scope.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3855
tools:
- name: Vlasiator
  role: Hybrid-Vlasov solver used to run the 1D collisionless shock benchmark at 1000
    km and 200 km resolutions, producing VLSV output files.
  repo: https://github.com/fmihpc/vlasiator
- name: analysator
  role: Python toolset for reading and analysing Vlasiator VLSV output files, used
    to extract compression ratios and ion distribution metrics.
  repo: https://github.com/fmihpc/analysator
- name: vlsv
  role: VLSV file format/library underlying Vlasiator output data files that downstream
    analysis reads.
  repo: https://github.com/fmihpc/vlsv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s41115-018-0003-2
    title: Vlasov methods in space physics and astrophysics
  source_content_hash: sha256:7f0179ccdbd9eb2851049e77027ee3a923e754608a6bfce108e6aa892cfdc311
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fmihpc/vlasiator
  tools:
  - Vlasiator
  - analysator
  - vlsv
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3855
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# collisionless-shock-initialization

## Summary

Configure and run a 1D collisionless shock benchmark in the Vlasiator hybrid-Vlasov solver under terrestrial bow-shock-like plasma conditions, at two grid resolutions (1000 km and 200 km), to test whether kinetic (non-fluid) shock physics is captured and quantify how refinement improves physical detail.

## When to use

Use when you need to reproduce a 1D collisionless-shock discretisation study with Vlasiator, e.g. to check whether a hybrid-Vlasov model departs from fluid (MHD) theory at coarse resolutions like ~1000 km.

## When NOT to use

- Input plasma is well described by single-fluid MHD and you only need the cheaper MHD-like answer; the point of this skill is to resolve departures from fluid theory, not to replicate MHD.
- The system of interest requires full global magnetospheric geometry (2D/3D magnetosphere coupling) rather than the 1D shock benchmark; this skill covers only the 1D normal-incidence-style shock setup.
- You need self-consistent electron kinetics — in Vlasiator electrons are treated as an MHD fluid, so purely electron-scale kinetic questions are out of scope.

## Inputs

- Vlasiator 1D shock benchmark configuration with terrestrial bow-shock-like plasma conditions (Pfau-Kempf et al. 2018 setup)
- Vlasiator solver binary built from github.com/fmihpc/vlasiator (with git submodules initialised)
- Grid resolution parameters (1000 km coarse run; 200 km refined run)

## Outputs

- Vlasiator VLSV output data files for the 1000 km and 200 km shock runs
- shock_comparison_table.csv with compression ratio and non-Maxwellianity fields for both resolutions
- Analysis of the downstream shock region (magnetic compression ratio, non-Maxwellianity of downstream ion distribution)

## How to apply

Compile and install Vlasiator from the fmihpc/vlasiator repository, cloning with --recurse-submodules so dependent header libraries are initialised. Configure the 1D collisionless shock benchmark following the Pfau-Kempf et al. (2018) setup with terrestrial bow-shock-like plasma conditions, in which ions are represented as velocity distribution functions while electrons are a magnetohydrodynamic fluid. Run the identical setup twice: first at the coarse 1000 km spatial resolution, then at the refined 200 km resolution, writing VLSV output files. Analyse the downstream shock region in both runs, extracting the magnetic compression ratio and computing a non-Maxwellianity metric for the downstream ion distribution, assembling results into shock_comparison_table.csv. Accept the run only if the compression ratio falls near the ~4x intensification reported for the terrestrial bow shock, required fields are present, and the 200 km run shows improved physical detail consistent with a kinetic description.

## Related tools

- **Vlasiator** (Hybrid-Vlasov solver used to run the 1D collisionless shock benchmark at 1000 km and 200 km resolutions, producing VLSV output files.) — https://github.com/fmihpc/vlasiator
- **analysator** (Python toolset for reading and analysing Vlasiator VLSV output files, used to extract compression ratios and ion distribution metrics.) — https://github.com/fmihpc/analysator
- **vlsv** (VLSV file format/library underlying Vlasiator output data files that downstream analysis reads.) — https://github.com/fmihpc/vlsv

## Examples

```
git clone --recurse-submodules https://github.com/fmihpc/vlasiator && git submodule update --init --recursive
```

## Evaluation signals

- Magnetic compression ratio values in the downstream region fall near the ~4x intensification reported for the terrestrial bow shock.
- Required fields are present in shock_comparison_table.csv for both the 1000 km and 200 km runs.
- The analysis script runs to completion without errors over the VLSV outputs.
- The 200 km run shows improved physical detail (better-resolved downstream structure and non-Maxwellianity signal) relative to the 1000 km run.
- Downstream ion distributions depart from fluid (MHD) theory, consistent with a kinetic description as confirmed by expert review.

## Limitations

- The hybrid-Vlasov approach has immense computational challenges due to the multi-dimensional ion-scale approach; even the 1D benchmark at 200 km resolution requires substantial resources, and global simulations need massively parallel HPC on tens of thousands of cores.
- Electrons are treated as a magnetohydrodynamic fluid, so electron-kinetic effects are not captured by this setup.
- At the coarse 1000 km resolution the physical detail is limited; resolution refinement to 200 km is needed to dramatically improve the accessible physical detail.

## Evidence

- [readme] Ions as VDFs vs electron fluid: [evidence span withheld — anchor status: external_doc]
- [readme] Install instructions: "git clone --recurse-submodules https://github.com/fmihpc/vlasiator

git checkout <branch>
git submodule update --init --recursive"
- [intro] Vlasiator prime application: "The world’s (so far) only global magnetospheric hybrid-Vlasov simulation, the massively parallel Vlasiator, is therefore the prime application in this article."
- [other] Analysator purpose: [evidence span withheld — anchor status: not_found]
- [readme] HPC parallelisation: [evidence span withheld — anchor status: external_doc]
