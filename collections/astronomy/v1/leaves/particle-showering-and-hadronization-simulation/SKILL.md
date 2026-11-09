---
name: particle-showering-and-hadronization-simulation
description: Use when a dark matter model predicts annihilation into a multitude of
  final states and continuum photon spectra dN/dE per annihilation are needed for
  gamma-ray likelihoods.
when_to_use_negative:
- The analysis needs only direct detection likelihoods or neutrino telescope likelihoods
  (gamLike/DDCalc/nulike), not photon yields
- A single simple annihilation channel is being studied where pre-tabulated yields
  from DarkSUSY or micrOMEGAs alone suffice, and the cascade-decay Monte Carlo machinery
  is unnecessary
- The particle cascade involves decays beyond the scope of Pythia 8's shower and hadronization
  models (e.g. purely non-hadronic processes)
tools:
- name: DarkBit
  role: Hosts the dedicated gamma-ray signal yield calculator implementing the cascade-decay
    Monte Carlo; accumulates photon energies into the requested bins
  repo: github.com/GambitBSM/gambit
- name: Pythia 8
  role: External backend used to shower and hadronize unstable annihilation final
    states, tracking cascade decays down to stable particles
- name: GAMBIT
  role: Framework that runs the scan via 'gambit -f <config.yaml>' and provides the
    backend system interfacing Pythia 8, DarkSUSY and micrOMEGAs
  repo: github.com/GambitBSM/gambit
- name: DarkSUSY
  role: Backend providing reference per-bin gamma-ray yields for cross-checking the
    Monte Carlo yield tables
- name: micrOMEGAs
  role: Backend providing reference per-bin gamma-ray yields for cross-checking the
    Monte Carlo yield tables
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1140/epjc/s10052-017-5155-4
    title: 'DarkBit: a GAMBIT module for computing dark matter observables and likelihoods'
  source_content_hash: sha256:dfe291de42bab67435e24a210137d0fd6ee02d6d1669cf5cc8867c1631846b27
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DarkBit
  - DarkSUSY
  - GAMBIT
  - Pythia 8
  - micrOMEGAs
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# particle-showering-and-hadronization-simulation

## Summary

Simulates the cascade of dark matter annihilation products down to stable particles by sampling hard final states, showering and hadronizing them with Pythia 8 inside DarkBit's cascade-decay Monte Carlo, to build per-final-state continuum gamma-ray yields dN/dE.

## When to use

Use when a dark matter model predicts annihilation into a multitude of final states and continuum photon spectra dN/dE per annihilation are needed for gamma-ray likelihoods.

## When NOT to use

- The analysis needs only direct detection likelihoods or neutrino telescope likelihoods (gamLike/DDCalc/nulike), not photon yields
- A single simple annihilation channel is being studied where pre-tabulated yields from DarkSUSY or micrOMEGAs alone suffice, and the cascade-decay Monte Carlo machinery is unnecessary
- The particle cascade involves decays beyond the scope of Pythia 8's shower and hadronization models (e.g. purely non-hadronic processes)

## Inputs

- GAMBIT YAML scan configuration specifying dark matter mass, annihilation final states to tabulate, and log-spaced photon energy binning
- Dark matter model annihilation final states (hard-process product definitions)
- Pythia 8 backend configuration for showering and hadronization

## Outputs

- Continuum gamma-ray yield tables dN/dE per annihilation, one column per requested final state and one row per log-spaced energy bin
- Per-bin Monte Carlo statistical uncertainties on each yield entry

## How to apply

Read the GAMBIT YAML scan configuration specifying the dark matter mass, the annihilation final states to tabulate, and the log-spaced photon energy binning, then launch DarkBit via 'gambit -f <config.yaml>'. For each annihilation final state, run DarkBit's cascade-decay Monte Carlo: sample the hard annihilation products, then shower and hadronize unstable final states with the interfaced Pythia 8 backend, tracking cascade decays down to stable particles. Accumulate the resulting photon energies into the requested log-spaced energy bins to build the continuum gamma-ray yield dN/dE per annihilation for each final state. Pythia 8 is attached through GAMBIT's 'backend' system, so the shower/hadronization step is delegated to the external code rather than reimplemented in DarkBit. Cross-check each final-state yield table against reference yields from the interfaced DarkSUSY and micrOMEGAs backends, mirroring the paper's quantitative dark-matter-code comparison.

## Related tools

- **DarkBit** (Hosts the dedicated gamma-ray signal yield calculator implementing the cascade-decay Monte Carlo; accumulates photon energies into the requested bins) — github.com/GambitBSM/gambit
- **Pythia 8** (External backend used to shower and hadronize unstable annihilation final states, tracking cascade decays down to stable particles)
- **GAMBIT** (Framework that runs the scan via 'gambit -f <config.yaml>' and provides the backend system interfacing Pythia 8, DarkSUSY and micrOMEGAs) — github.com/GambitBSM/gambit
- **DarkSUSY** (Backend providing reference per-bin gamma-ray yields for cross-checking the Monte Carlo yield tables)
- **micrOMEGAs** (Backend providing reference per-bin gamma-ray yields for cross-checking the Monte Carlo yield tables)

## Examples

```
gambit -f yaml_files/MSSM7.yaml
```

## Evaluation signals

- Every DarkBit yield entry lies within the Monte Carlo statistical uncertainty of the corresponding DarkSUSY/micrOMEGAs reference per-bin yield
- The output yield table contains one yield column per requested final state and one row per requested log-spaced energy bin
- Each cascade chain terminates in stable particles only (unstable states are fully showered/hadronized by Pythia 8)
- Photon energies are correctly accumulated into the log-spaced bins specified in the YAML configuration

## Limitations

- Yields carry Monte Carlo statistical uncertainty, so single-bin comparisons against reference codes must account for this spread
- Agreement is only as good as the Pythia 8 shower/hadronization modeling and the backend configurations of DarkSUSY and micrOMEGAs
- Building the full stack (gambit plus backends including Pythia 8) requires the CMake build and 'make -jn backends'; verify with 'gambit backends' before running

## Evidence

- [abstract] DarkBit gamma-ray yield calculator: "a dedicated signal yield calculator for gamma-ray observations, which significantly extends current tools by implementing a cascade-decay Monte Carlo"
- [intro] Backend system: "a flexible system for interfacing external codes (the "backend" system)"
- [methods] Run GAMBIT scan: "gambit -f yaml_files/MSSM7.yaml"
- [readme] Pythia 8 dependency: "**Pythia 8:** **(i)** T. Sjostrand, S. Ask, et al., An Introduction to PYTHIA 8.2, Comp. Phys. Comm. 191 (2015) 159"
- [methods] Verify backends: "gambit backends
  gambit scanners"
