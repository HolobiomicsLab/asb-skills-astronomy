---
name: cross-code-yield-comparison
description: Use when a newly computed dN/dE photon yield table for dark matter annihilation
  final states must be validated against established codes, or when quantitatively
  comparing dark matter codes' indirect-detection outputs.
when_to_use_negative:
- The analysis concerns direct detection likelihoods (spin-independent/spin-dependent
  scattering) rather than gamma-ray photon yields
- No independent reference code (DarkSUSY or micrOMEGAs) is available or interfaced,
  so no external cross-check can be performed
- The model's annihilation involves few or exotic final states where the reference
  codes lack tabulated branching yields, so a meaningful comparison is impossible
edam_operation: http://edamontology.org/operation_2424
edam_topics: []
tools:
- name: DarkBit
  role: 'Code under test: computes gamma-ray signal yields via its cascade-decay Monte
    Carlo, whose output tables are cross-checked'
  repo: github.com/GambitBSM/gambit
- name: DarkSUSY
  role: Reference backend providing independent per-bin gamma-ray yields for comparison
- name: micrOMEGAs
  role: Reference backend providing independent per-bin gamma-ray yields for comparison
- name: Pythia 8
  role: Interfaced backend used by DarkBit's cascade-decay Monte Carlo for showering
    and hadronization of unstable final states
- name: GAMBIT
  role: Framework in which DarkBit runs the scan configured by the YAML input file
  repo: github.com/GambitBSM/gambit
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1140/epjc/s10052-017-5155-4
    title: 'DarkBit: a GAMBIT module for computing dark matter observables and likelihoods'
  source_content_hash: sha256:93e4e355992cbb4f84a98f346579cc280d313525a733b338bf49e1241f419ba9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DarkBit
  - DarkSUSY
  - GAMBIT
  - Pythia 8
  - micrOMEGAs
  edam_operation: http://edamontology.org/operation_2424
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# cross-code-yield-comparison

## Summary

Validates a dark-matter code's gamma-ray yield tables by cross-checking them against yields computed with independent reference codes (DarkSUSY and micrOMEGAs), mirroring the DarkBit paper's quantitative dark-matter-code comparison.

## When to use

Use when a newly computed dN/dE photon yield table for dark matter annihilation final states must be validated against established codes, or when quantitatively comparing dark matter codes' indirect-detection outputs.

## When NOT to use

- The analysis concerns direct detection likelihoods (spin-independent/spin-dependent scattering) rather than gamma-ray photon yields
- No independent reference code (DarkSUSY or micrOMEGAs) is available or interfaced, so no external cross-check can be performed
- The model's annihilation involves few or exotic final states where the reference codes lack tabulated branching yields, so a meaningful comparison is impossible

## Inputs

- YAML GAMBIT scan configuration specifying dark matter mass, annihilation final states to tabulate, and log-spaced photon energy binning
- dN/dE per-bin photon yield table from the code under test (one column per final state, one row per energy bin)
- Reference per-bin yields from DarkSUSY backend
- Reference per-bin yields from micrOMEGAs backend

## Outputs

- Validated yield table (one yield column per requested final state, one row per energy bin) accepted only within Monte Carlo statistical uncertainty of reference codes
- Per-bin comparison of yields across DarkBit, DarkSUSY, and micrOMEGAs for each annihilation final state

## How to apply

For each requested annihilation final state, compute the continuum gamma-ray yield dN/dE per annihilation with the code under test (DarkBit's cascade-decay Monte Carlo: sample hard annihilation products, shower and hadronize with Pythia 8, accumulate photon energies into log-spaced bins), then recompute the same per-bin yields with the interfaced DarkSUSY and micrOMEGAs backends. Tabulate yields with one column per final state and one row per photon energy bin, using the same log-spaced energy binning and dark matter mass across all three codes. Accept the table under test only if every entry lies within the Monte Carlo statistical uncertainty of the DarkSUSY/micrOMEGAs per-bin reference yields; discrepancies beyond statistical error indicate a showering, hadronization, or decay-cascade bug. This mirrors the DarkBit paper's quantitative comparison between DarkSUSY and micrOMEGAs, which is the stated rationale for implementing an independent cascade-decay Monte Carlo.

## Related tools

- **DarkBit** (Code under test: computes gamma-ray signal yields via its cascade-decay Monte Carlo, whose output tables are cross-checked) — github.com/GambitBSM/gambit
- **DarkSUSY** (Reference backend providing independent per-bin gamma-ray yields for comparison)
- **micrOMEGAs** (Reference backend providing independent per-bin gamma-ray yields for comparison)
- **Pythia 8** (Interfaced backend used by DarkBit's cascade-decay Monte Carlo for showering and hadronization of unstable final states)
- **GAMBIT** (Framework in which DarkBit runs the scan configured by the YAML input file) — github.com/GambitBSM/gambit

## Examples

```
gambit -f config.yaml && gambit backends
```

## Evaluation signals

- Every DarkBit per-bin yield entry lies within the Monte Carlo statistical uncertainty of the corresponding DarkSUSY/micrOMEGAs reference yield
- The yield table contains exactly one yield column per requested final state and one row per log-spaced energy bin
- All three codes (DarkBit, DarkSUSY, micrOMEGAs) were run with identical dark matter mass, final-state list, and photon energy binning
- Backends were built and verified as installed via 'gambit backends' before the comparison was run

## Limitations

- The cascade-decay Monte Carlo carries statistical uncertainty, so agreement is judged per-bin within that uncertainty rather than exactly
- Validation depends on Pythia 8 backend availability and correct build of DarkSUSY and micrOMEGAs backends within GAMBIT

## Evidence

- [abstract] DarkBit includes a dedicated gamma-ray signal yield calculator implementing a cascade-decay Monte Carlo: "a dedicated signal yield calculator for gamma-ray observations, which significantly extends current tools by implementing a cascade-decay Monte Carlo"
- [abstract] The paper includes a quantitative comparison between DarkSUSY and micrOMEGAs: "a quantitative comparison between two of the main dark matter codes ( DarkSUSY and micrOMEGAs )"
- [methods] Verify installed backends and scanners before running comparison: "gambit backends
  gambit scanners"
- [methods] Run GAMBIT scan via YAML input file: "gambit -f yaml_files/MSSM7.yaml"
- [intro] GAMBIT backend system interfaces external codes such as DarkSUSY and micrOMEGAs: "a flexible system for interfacing external codes (the "backend" system)"
