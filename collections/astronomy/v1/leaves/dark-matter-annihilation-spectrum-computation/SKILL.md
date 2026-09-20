---
name: dark-matter-annihilation-spectrum-computation
description: Use when a dark matter model predicts annihilation into multiple final
  states and per-final-state gamma-ray yields dN/dE on a log-spaced photon energy
  grid are needed for indirect-detection likelihoods.
when_to_use_negative:
- The goal is direct detection constraints or neutrino telescope likelihoods — use
  DarkBit's DDCalc or nulike functionality instead of the gamma-ray yield calculator
- The model has a single, simple annihilation final state already covered by standard
  DarkSUSY/micrOMEGAs routines, so the cascade-decay Monte Carlo extension is unnecessary
edam_operation: http://edamontology.org/operation_2945
tools:
- name: DarkBit
  role: GAMBIT module hosting the dedicated gamma-ray signal yield calculator with
    the cascade-decay Monte Carlo
  repo: github.com/GambitBSM/gambit
- name: GAMBIT
  role: Framework that launches the scan from the YAML config ('gambit -f <config.yaml>')
    and supplies the backend system for external codes
  repo: github.com/GambitBSM/gambit
- name: Pythia 8
  role: Interfaced backend used to shower and hadronize unstable final states and
    track cascade decays to stable particles
- name: DarkSUSY
  role: Interfaced backend providing reference per-bin yields for cross-checking DarkBit's
    yield tables
- name: micrOMEGAs
  role: Interfaced backend providing reference per-bin yields for cross-checking DarkBit's
    yield tables
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1140/epjc/s10052-017-5155-4
    title: 'DarkBit: a GAMBIT module for computing dark matter observables and likelihoods'
  source_content_hash: sha256:1688a9d285568f99bce523fa991b8ee23091a513b9dc9ad7985e9ae4403970ba
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DarkBit
  - DarkSUSY
  - GAMBIT
  - Pythia 8
  - micrOMEGAs
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# dark-matter-annihilation-spectrum-computation

## Summary

Computes continuum gamma-ray photon signal yields dN/dE per annihilation for dark matter models with many final states, using DarkBit's cascade-decay Monte Carlo inside GAMBIT, with Pythia 8 showering/hadronization and cross-validation against DarkSUSY and micrOMEGAs.

## When to use

Use when a dark matter model predicts annihilation into multiple final states and per-final-state gamma-ray yields dN/dE on a log-spaced photon energy grid are needed for indirect-detection likelihoods.

## When NOT to use

- The goal is direct detection constraints or neutrino telescope likelihoods — use DarkBit's DDCalc or nulike functionality instead of the gamma-ray yield calculator
- The model has a single, simple annihilation final state already covered by standard DarkSUSY/micrOMEGAs routines, so the cascade-decay Monte Carlo extension is unnecessary

## Inputs

- YAML GAMBIT scan configuration specifying dark matter mass, annihilation final states to tabulate, and log-spaced photon energy binning
- Dark matter model (e.g. a simple effective dark matter model) with annihilation channels into many final states

## Outputs

- Gamma-ray yield table dN/dE per annihilation, with one yield column per requested final state and one row per log-spaced energy bin
- Per-bin Monte Carlo statistical uncertainties on the DarkBit yields for comparison against DarkSUSY/micrOMEGAs reference yields

## How to apply

Read a YAML scan configuration specifying the dark matter mass, the annihilation final states to tabulate, and the log-spaced photon energy binning, then launch DarkBit within GAMBIT (e.g. 'gambit -f <config.yaml>'). For each annihilation final state, run DarkBit's cascade-decay Monte Carlo: sample the hard annihilation products, then shower and hadronize unstable final states with the interfaced Pythia 8 backend, tracking cascade decays down to stable particles. Accumulate the resulting photon energies into the requested energy bins to build the continuum gamma-ray yield dN/dE per annihilation for each final state, producing one yield column per requested final state and one row per energy bin. Cross-check each final-state yield table against reference yields from the interfaced DarkSUSY and micrOMEGAs backends, exploiting GAMBIT's backend system that interfaces external codes. Accept the yield table only if every DarkBit entry lies within the Monte Carlo statistical uncertainty of the DarkSUSY/micrOMEGAs reference per-bin yields.

## Related tools

- **DarkBit** (GAMBIT module hosting the dedicated gamma-ray signal yield calculator with the cascade-decay Monte Carlo) — github.com/GambitBSM/gambit
- **GAMBIT** (Framework that launches the scan from the YAML config ('gambit -f <config.yaml>') and supplies the backend system for external codes) — github.com/GambitBSM/gambit
- **Pythia 8** (Interfaced backend used to shower and hadronize unstable final states and track cascade decays to stable particles)
- **DarkSUSY** (Interfaced backend providing reference per-bin yields for cross-checking DarkBit's yield tables)
- **micrOMEGAs** (Interfaced backend providing reference per-bin yields for cross-checking DarkBit's yield tables)

## Examples

```
gambit -f yaml_files/MSSM7.yaml
```

## Evaluation signals

- Every DarkBit yield entry lies within the Monte Carlo statistical uncertainty of the DarkSUSY/micrOMEGAs reference per-bin yield
- The output table contains exactly one yield column per requested final state and one row per requested energy bin
- Energy bins follow the log-spaced photon energy binning specified in the YAML configuration
- All requested annihilation final states tabulated in the YAML config appear in the output yield table

## Limitations

- Yields are Monte Carlo estimates, so per-bin statistical uncertainty must be propagated when comparing against DarkSUSY/micrOMEGAs reference yields
- Complex models with annihilation to many final states are the target of this extension; single-channel yields may be obtained more directly from DarkSUSY or micrOMEGAs

## Evidence

- [abstract] cascade-decay Monte Carlo extends current gamma-ray yield tools: "a dedicated signal yield calculator for gamma-ray observations, which significantly extends current tools by implementing a cascade-decay Monte Carlo"
- [abstract] quantitative DarkSUSY vs micrOMEGAs comparison: "a quantitative comparison between two of the main dark matter codes ( DarkSUSY and micrOMEGAs )"
- [intro] GAMBIT backend system interfaces external codes: "a flexible system for interfacing external codes (the "backend" system)"
- [methods] launch GAMBIT scan from YAML input file: "gambit -f yaml_files/MSSM7.yaml"
