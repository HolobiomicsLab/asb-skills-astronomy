---
name: condensation-temperature-simulation
description: Use when you need condensation temperatures (Tc) of planet-building elements
  for a stellar abundance pattern, or when quantifying how ∆Tc responds to varying
  disk pressure, metallicity, or element ratios like C/O from 0.1 to 0.7.
when_to_use_negative:
- You need gas-phase-only chemistry without condensation — the TEA code (limited to
  gas-phase simulations) is cited as the appropriate alternative for that scope
- The stellar abundance data is very incomplete (e.g. Al, Ca, Mg, Si, or O data is
  not reported) — the README warns to consider the sensibility of running simulations
  on such data
- You require solid-solution mineralogy — the current code considers only pure minerals,
  with solid solutions planned for a later release
edam_operation: http://edamontology.org/operation_0242
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: ECCOplanets
  role: Open-source Python code performing the Gibbs free energy minimisation condensation
    simulation and providing thermochemical and stellar abundance databases; samples
    s1simulation.py and s2analyses.py drive the workflow
  repo: https://github.com/AninaTimmermann/ECCOplanets
- name: Python (pandas, numpy, matplotlib)
  role: Runtime and packages for running the simulation, tabulating Tc/∆Tc, and plotting
    trends
  repo: https://github.com/AninaTimmermann/ECCOplanets
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1051/0004-6361/202244850
    title: Revisiting equilibrium condensation and rocky planet compositions
  source_content_hash: sha256:9260814cd7c2e686134a9161fb2b309d3c2febae2699bedab087c1d76604d739
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AninaTimmermann/ECCOplanets
  tools:
  - ECCOplanets
  - Python
  - Python (pandas, numpy, matplotlib)
  - matplotlib
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_0242
  edam_topics:
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# condensation-temperature-simulation

## Summary

Run Gibbs free energy minimisation condensation simulations with ECCOplanets to derive temperature-dependent condensation temperatures (Tc) of elements and molecules for a given stellar abundance pattern and disk pressure, and quantify their sensitivity (∆Tc) to variations in key element ratios such as C/O.

## When to use

Use when you need condensation temperatures (Tc) of planet-building elements for a stellar abundance pattern, or when quantifying how ∆Tc responds to varying disk pressure, metallicity, or element ratios like C/O from 0.1 to 0.7.

## When NOT to use

- You need gas-phase-only chemistry without condensation — the TEA code (limited to gas-phase simulations) is cited as the appropriate alternative for that scope
- The stellar abundance data is very incomplete (e.g. Al, Ca, Mg, Si, or O data is not reported) — the README warns to consider the sensibility of running simulations on such data
- You require solid-solution mineralogy — the current code considers only pure minerals, with solid solutions planned for a later release

## Inputs

- ECCOplanets thermochemical database (.pkl files in the 'data' folder, loaded via data.py)
- Stellar abundance data (Brewer+2016 F, G, and K star database, .pkl)
- Simulation parameters: temperature range and resolution, molecule selection, pressure, elemental abundance pattern (set in s1simulation.py)

## Outputs

- Simulation results (element/molecule amounts vs. temperature, stored in 'simulations' folder)
- Condensation temperatures (Tc) of common planet-building elements for each abundance setting
- ∆Tc table versus C/O (e.g. condensation_temperatures_co_variation.csv) and matplotlib trend plots

## How to apply

Load the thermochemical databases and stellar abundance data (.pkl files) via data.py, then set the condensation simulation parameters in s1simulation.py: temperature range and resolution, molecule selection, and the elemental abundance pattern. To test abundance-pattern sensitivity, adapt carbon and oxygen abundances following the parabolic correlation of C/O with Sigma/O from the Brewer+2016 stellar sample, generating patterns spanning C/O = 0.1 to 0.7 while keeping all other element ratios fixed, and rerun the Gibbs free energy minimisation (simulation.py) with unchanged pressure and molecule selection. After each run, use s2analyses.py (classes.py) to load the simulation, analyse condensation of molecules and elements, and improve results by cropping the gas phase, recomputing glitched sections, removing numerical glitches, and smoothing the curve. Extract Tc for the common planet-building elements per setting and compute ∆Tc relative to a baseline (e.g. C/O = 0.1); tabulate and plot the trends with matplotlib/pandas. Validate against known benchmarks: solar-parameter Tc within ±5 K of literature values, pressure range 10^-6–0.1 bar giving ∆Tc ≈ +350 K, and C/O 0.1→0.7 giving ∆Tc ≈ −100 K.

## Related tools

- **ECCOplanets** (Open-source Python code performing the Gibbs free energy minimisation condensation simulation and providing thermochemical and stellar abundance databases; samples s1simulation.py and s2analyses.py drive the workflow) — https://github.com/AninaTimmermann/ECCOplanets
- **Python (pandas, numpy, matplotlib)** (Runtime and packages for running the simulation, tabulating Tc/∆Tc, and plotting trends) — https://github.com/AninaTimmermann/ECCOplanets

## Examples

```
python s1simulation.py  # set temperature range/resolution, molecule selection, abundance pattern (C/O 0.1–0.7); then python s2analyses.py
```

## Evaluation signals

- Solar-parameter condensation temperatures of common planet-building elements fall within ±5 K of literature values
- Increasing C/O from 0.1 to 0.7 yields ∆Tc ≈ −100 K; other element ratios show smaller effects
- Pressure variation over 10^-6–0.1 bar yields ∆Tc ≈ +350 K and −0.3 ≤ [M/H] ≤ 0.4 yields ∆Tc ≈ +100 K for most elements
- Simulation improvements (cropping gas phase, recomputing a section, glitch removal, smoothing) visibly remove numerical glitches and smooth the amount-vs-temperature curves
- Output table (e.g. condensation_temperatures_co_variation.csv) contains Tc and ∆Tc for every tested C/O value relative to the C/O = 0.1 baseline

## Limitations

- Only pure minerals are considered at the moment; solid solutions are planned for a later point
- First release only includes abundance data of F, G, and K stars from the Brewer+2016 database; other stellar types can be added but uncertainty must be considered
- Running simulations on very incomplete abundance data sets (e.g. missing Al, Ca, Mg, Si, or O) may not be sensible
- The code is a very simplified starting point to get an approximate idea of planetary composition variety, not a high-fidelity model

## Evidence

- [abstract] Gibbs minimisation simulation setup: "Our T-p-dependent chemical equilibrium model is based on a Gibbs free energy minimisation."
- [abstract] C/O sensitivity finding: "An increase in C/O from 0.1 to 0.7 results in a decrease of ∆Tc ≈ −100 K. Other element ratios are less influential."
- [readme] Code stack: "The code is built in python and uses the common packages pandas, numpy and matplotlib."
- [readme] Simulation sample description: "In this sample we introduce the parameters of the condensation simulation, i.e. the temperature range and resolution, the molecule selection, and the elemental abundance pattern."
- [readme] Result improvement steps: "improving the simulation result by recomputing a section, removing numerical glitches, and smoothing the curve"
- [readme] Data loading: "this code only reads the .pkl files containing all relevant data into python"
- [readme] Brewer+2016 scope: "This first release only includes abundance data of F, G, and K stars from the Brewer+2016 database."
- [abstract] Validation benchmark: "For common planet-building elements, we derive a Tc that is within ±5 K of literature values"
