---
name: cascade-decay-monte-carlo-sampling
description: Use when a dark matter model annihilates into a multitude of final states
  and continuum gamma-ray yields dN/dE per final state are needed on log-spaced photon
  energy bins.
when_to_use_negative:
- The task is direct detection or neutrino telescope likelihood computation — those
  are handled by DarkBit's DDCalc and nulike/gamLike likelihood machinery, not the
  cascade-decay Monte Carlo
- Only a single simple annihilation channel is needed and a validated reference code
  (DarkSUSY or micrOMEGAs) already provides the yield — the Monte Carlo is aimed at
  extending current tools for complex models with many final states
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: DarkBit
  role: GAMBIT module hosting the dedicated gamma-ray signal yield calculator that
    implements the cascade-decay Monte Carlo
  repo: github.com/GambitBSM/gambit
- name: GAMBIT
  role: Framework in which DarkBit is launched via a YAML scan configuration
  repo: github.com/GambitBSM/gambit_1.5
- name: Pythia 8
  role: Interfaced backend used to shower and hadronize unstable final states and
    track cascade decays down to stable particles
- name: DarkSUSY
  role: Interfaced backend providing reference per-bin gamma-ray yields for cross-validation
- name: micrOMEGAs
  role: Interfaced backend providing reference per-bin gamma-ray yields for cross-validation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1140/epjc/s10052-017-5155-4
    title: 'DarkBit: a GAMBIT module for computing dark matter observables and likelihoods'
  source_content_hash: sha256:47721cdf7548bc62abb281b5eaecad14b12881487aa93536e4c8eb0b807f64a5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DarkBit
  - DarkSUSY
  - GAMBIT
  - Pythia 8
  - micrOMEGAs
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cascade-decay-monte-carlo-sampling

## Summary

Computes gamma-ray photon signal yields dN/dE per annihilation for dark matter models with many final states by running DarkBit's dedicated cascade-decay Monte Carlo, which showers and hadronizes hard annihilation products with the Pythia 8 backend and bins the resulting photon energies.

## When to use

Use when a dark matter model annihilates into a multitude of final states and continuum gamma-ray yields dN/dE per final state are needed on log-spaced photon energy bins.

## When NOT to use

- The task is direct detection or neutrino telescope likelihood computation — those are handled by DarkBit's DDCalc and nulike/gamLike likelihood machinery, not the cascade-decay Monte Carlo
- Only a single simple annihilation channel is needed and a validated reference code (DarkSUSY or micrOMEGAs) already provides the yield — the Monte Carlo is aimed at extending current tools for complex models with many final states

## Inputs

- YAML GAMBIT scan configuration specifying dark matter mass, annihilation final states to tabulate, and log-spaced photon energy binning
- Dark matter model predicting annihilation into a multitude of final states

## Outputs

- Continuum gamma-ray yield table dN/dE per annihilation, one yield column per requested final state and one row per energy bin
- Monte Carlo statistical uncertainties on per-bin yields

## How to apply

Read the YAML scan configuration specifying the dark matter mass, the annihilation final states to tabulate, and the log-spaced photon energy binning, then launch DarkBit within GAMBIT. For each annihilation final state, run DarkBit's cascade-decay Monte Carlo: sample the hard annihilation products, then shower and hadronize unstable final states with the interfaced Pythia 8 backend, tracking cascade decays down to stable particles. Accumulate the resulting photon energies into the requested energy bins to build the continuum gamma-ray yield dN/dE per annihilation for each final state. Cross-check each final-state yield table against reference yields computed with the interfaced DarkSUSY and micrOMEGAs backends, mirroring the paper's quantitative dark-matter-code comparison. Accept the yield table only if every DarkBit entry lies within the Monte Carlo statistical uncertainty of the DarkSUSY/micrOMEGAs per-bin reference yields, with one yield column per requested final state and one row per energy bin.

## Related tools

- **DarkBit** (GAMBIT module hosting the dedicated gamma-ray signal yield calculator that implements the cascade-decay Monte Carlo) — github.com/GambitBSM/gambit
- **GAMBIT** (Framework in which DarkBit is launched via a YAML scan configuration) — github.com/GambitBSM/gambit_1.5
- **Pythia 8** (Interfaced backend used to shower and hadronize unstable final states and track cascade decays down to stable particles)
- **DarkSUSY** (Interfaced backend providing reference per-bin gamma-ray yields for cross-validation)
- **micrOMEGAs** (Interfaced backend providing reference per-bin gamma-ray yields for cross-validation)

## Examples

```
gambit -f <config.yaml>
```

## Evaluation signals

- Every DarkBit per-bin yield entry lies within the Monte Carlo statistical uncertainty of the DarkSUSY/micrOMEGAs reference yield for that bin
- The output table contains one yield column per requested final state and one row per energy bin
- Energy binning matches the log-spaced photon energy bins specified in the YAML configuration
- The yield table reports dN/dE per annihilation (normalized per annihilation event) for each final state
- Cross-checks reproduce the paper's quantitative comparison between DarkSUSY and micrOMEGAs

## Limitations

- Yields carry Monte Carlo statistical uncertainty, so agreement with reference codes is only validated within that uncertainty rather than exactly
- The calculator targets complex models with annihilation to many final states; for such complex models it significantly extends current tools, but single-channel use may be redundant with DarkSUSY/micrOMEGAs

## Evidence

- [abstract] a dedicated signal yield calculator for gamma-ray observations, which significantly extends current tools by implementing a cascade-decay Monte Carlo: "a dedicated signal yield calculator for gamma-ray observations, which significantly extends current tools by implementing a cascade-decay Monte Carlo"
- [abstract] DarkBit is written in the framework of the Global And Modular Beyond the Standard Model Inference Tool ( GAMBIT ): "DarkBit is written in the framework of the Global And Modular Beyond the Standard Model Inference Tool ( GAMBIT )"
- [abstract] along with interfaces to DarkSUSY , micrOMEGAs , and the neutrino telescope likelihood package nulike: "along with interfaces to DarkSUSY , micrOMEGAs , and the neutrino telescope likelihood package nulike"
- [abstract] a quantitative comparison between two of the main dark matter codes ( DarkSUSY and micrOMEGAs ): "a quantitative comparison between two of the main dark matter codes ( DarkSUSY and micrOMEGAs )"
- [methods] Run GAMBIT scan via yaml input file: gambit -f yaml_files/MSSM7.yaml: "gambit -f yaml_files/MSSM7.yaml"
