---
name: stellar-abundance-pattern-scaling
description: Use when you need to test how condensation temperatures (Tc) of planet-building
  elements respond to a changed disk elemental abundance pattern — e.g. an overall
  [M/H] offset or one key element ratio (C/O) at a time — using ECCOplanets.
when_to_use_negative:
- You are studying the effect of disk pressure instead of abundance pattern — pressure
  variation (10^-6–0.1 bar, ∆Tc ≈ +350 K) is a separate parameter sweep in the paper
- You need solid-solution minerals or complete abundance data for stellar types outside
  F, G, and K — the code considers only pure minerals and its first release ships
  Brewer+2016 F/G/K data only
- Stellar abundance data is missing key rock-forming elements (Al, Ca, Mg, Si, or
  O), which the README flags as questionable input for condensation simulations
tools:
- name: ECCOplanets
  role: Condensation-simulation framework providing data.py (.pkl loaders), s1simulation.py
    simulation parameters, and s2analyses.py result post-processing for the scaled
    abundance patterns
  repo: https://github.com/AninaTimmermann/ECCOplanets
- name: simulation.py
  role: Backbone implementing the Gibbs free energy minimisation run for each scaled
    abundance pattern
  repo: https://github.com/AninaTimmermann/ECCOplanets
- name: classes.py
  role: Simulation class functions used to analyse condensation of molecules and elements
    and clean the curves (crop gas phase, recompute sections, smooth)
  repo: https://github.com/AninaTimmermann/ECCOplanets
- name: pandas
  role: Loads the .pkl data files and holds the Tc/∆Tc table saved as condensation_temperatures_co_variation.csv
- name: matplotlib
  role: Plots Tc and ∆Tc trends versus [M/H] or C/O
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1051/0004-6361/202244850
    title: Revisiting equilibrium condensation and rocky planet compositions
  source_content_hash: sha256:8b93b649303f8cfcaa48f27d6185f87f61275b62b0b1b6d0462def4891acb42a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AninaTimmermann/ECCOplanets
  tools:
  - ECCOplanets
  - Python
  - classes.py
  - matplotlib
  - numpy
  - pandas
  - simulation.py
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# stellar-abundance-pattern-scaling

## Summary

Generates synthetic stellar/disk elemental abundance patterns by scaling the solar abundance pattern to metallicity offsets (-0.3 to +0.4 dex) or to a varied C/O ratio (0.1–0.7), then feeds them into the ECCOplanets Gibbs free energy minimisation condensation simulation to quantify ∆Tc sensitivity of planet-building elements.

## When to use

Use when you need to test how condensation temperatures (Tc) of planet-building elements respond to a changed disk elemental abundance pattern — e.g. an overall [M/H] offset or one key element ratio (C/O) at a time — using ECCOplanets.

## When NOT to use

- You are studying the effect of disk pressure instead of abundance pattern — pressure variation (10^-6–0.1 bar, ∆Tc ≈ +350 K) is a separate parameter sweep in the paper
- You need solid-solution minerals or complete abundance data for stellar types outside F, G, and K — the code considers only pure minerals and its first release ships Brewer+2016 F/G/K data only
- Stellar abundance data is missing key rock-forming elements (Al, Ca, Mg, Si, or O), which the README flags as questionable input for condensation simulations

## Inputs

- Solar elemental abundance pattern (fixed reference pattern used by the simulation class)
- ECCOplanets .pkl thermochemical data and stellar abundance databases read via data.py
- Brewer+2016 F, G, and K star abundance data (for C/O-guided pattern construction)
- List of common planet-building elements for which 50% condensation temperatures are extracted

## Outputs

- Condensation temperatures (Tc) of planet-building elements for each abundance-pattern variant
- Delta-Tc table relative to the solar-composition baseline (e.g. condensation_temperatures_co_variation.csv)
- Matplotlib plots of Tc and ∆Tc versus [M/H] or C/O

## How to apply

Load the thermochemical and stellar abundance .pkl databases via data.py, then generate synthetic abundance patterns either by scaling the solar pattern to metallicity offsets spanning -0.3 to +0.4 dex while keeping relative element ratios unchanged, or by adjusting C and O to span C/O = 0.1–0.7 following the parabolic C/O–Sigma/O correlation in the Brewer+2016 stellar sample while holding other ratios fixed. For each pattern, instantiate the simulation class as in s1simulation.py, keeping temperature range, resolution, molecule selection, and pressure unchanged so that ∆Tc reflects only the abundance-pattern change. Run the Gibbs free energy minimisation in simulation.py, then post-process with s2analyses.py/classes.py operations — cropping the gas phase, recomputing glitched sections, removing numerical glitches, and smoothing the curve — before extracting the 50% condensation temperatures of the common planet-building elements. Compute ∆Tc relative to the solar-composition (or C/O = 0.1) baseline and tabulate/plot Tc and ∆Tc versus the varied parameter. Rationale: varying one parameter at a time with everything else fixed isolates the abundance-pattern effect, which the paper reports as roughly +100 K over -0.3 ≤ [M/H] ≤ 0.4 and roughly -100 K for C/O rising from 0.1 to 0.7.

## Related tools

- **ECCOplanets** (Condensation-simulation framework providing data.py (.pkl loaders), s1simulation.py simulation parameters, and s2analyses.py result post-processing for the scaled abundance patterns) — https://github.com/AninaTimmermann/ECCOplanets
- **simulation.py** (Backbone implementing the Gibbs free energy minimisation run for each scaled abundance pattern) — https://github.com/AninaTimmermann/ECCOplanets
- **classes.py** (Simulation class functions used to analyse condensation of molecules and elements and clean the curves (crop gas phase, recompute sections, smooth)) — https://github.com/AninaTimmermann/ECCOplanets
- **pandas** (Loads the .pkl data files and holds the Tc/∆Tc table saved as condensation_temperatures_co_variation.csv)
- **matplotlib** (Plots Tc and ∆Tc trends versus [M/H] or C/O)

## Examples

```
python data.py; python s1simulation.py  # scale solar abundance pattern to [M/H] offsets (-0.3..+0.4) or C/O 0.1–0.7, then run Gibbs minimisation and analyse via s2analyses.py
```

## Evaluation signals

- Average ∆Tc over -0.3 ≤ [M/H] ≤ 0.4 is approximately +100 K for most elements, matching the paper's reported value
- Simulated ∆Tc for the C/O increase from 0.1 to 0.7 matches the reported decrease of approximately -100 K
- Baseline solar-composition condensation temperatures match literature values within ±5 K
- Only one abundance parameter varies at a time: relative element ratios are unchanged for [M/H] scaling, and all non-C/O element ratios are fixed in the C/O sweep
- Simulation parameters (temperature range, resolution, molecule selection, pressure) are identical across all runs so ∆Tc is attributable solely to the abundance pattern

## Limitations

- The first release only includes abundance data of F, G, and K stars from the Brewer+2016 database; other stellar types require care with uncertainties
- Running simulations on very incomplete abundance sets (missing Al, Ca, Mg, Si, or O) may be scientifically insensible
- The code considers only pure minerals at the moment; solid solutions are planned but not yet included
- Numerical glitches in condensation curves require post-processing (cropping the gas phase, recomputing a section, removing glitches, smoothing) before Tc extraction

## Evidence

- [abstract] Metallicity scaling finding: "The tested pressure range (10−6 − 0.1 bar) corresponds to ∆Tc ≈ +350 K, and for −0.3 ≤ [M/H] ≤ 0.4 we find ∆Tc ≈ +100 K"
- [abstract] C/O variation finding: "An increase in C/O from 0.1 to 0.7 results in a decrease of ∆Tc ≈ −100 K. Other element ratios are less influential."
- [methods] Simulation parameter setup: "In this sample we introduce the parameters of the condensation simulation, i.e. the temperature range and resolution, the molecule selection, and the elemental abundance pattern."
- [abstract] Accuracy benchmark: "For common planet-building elements, we derive a Tc that is within ±5 K of literature values"
- [readme] Data loading mechanism: "this code only reads the .pkl files containing all relevant data into python"
- [readme] Abundance database scope: "This first release only includes abundance data of F, G, and K stars from the Brewer+2016 database."
