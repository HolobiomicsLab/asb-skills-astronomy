---
name: gibbs-free-energy-minimisation-condensation-simulation
description: Use when you need 50% condensation temperatures (Tc) or ∆Tc of major
  rock-forming elements (e.g. Mg, Si, Fe, Al, Ca, Na, Ni) under a given stellar abundance
  pattern, or when quantifying how Tc shifts with disk pressure (e.g. 10^-6–0.1 bar)
  or metallicity.
when_to_use_negative:
- You need gas-phase-only equilibrium chemistry — the paper notes the TEA code is
  'limited to gas-phase simulations', so a gas-phase code is more appropriate if no
  condensation is involved
- Your abundance data is very incomplete (e.g. Al, Ca, Mg, Si, or O not reported)
  — the README cautions to consider 'the sensibility of running simulations on very
  incomplete abundance data sets'
- You need solid-solution mineralogy — 'We only consider pure minerals at the moment',
  so mixed-crystal equilibria require another tool
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_3318
tools:
- name: ECCOplanets
  role: Core open-source Python code that performs the Gibbs free energy minimisation
    condensation simulation and provides the thermochemical and stellar abundance
    databases
  repo: https://github.com/AninaTimmermann/ECCOplanets
- name: Python
  role: Language in which the code is built; simulation.py holds the mathematical
    and thermochemical formulation, and analysis uses pandas, numpy, matplotlib
- name: pandas
  role: Data handling for simulation curves, Tc extraction, and ∆Tc result tables
- name: numpy
  role: Numerical computation for condensation curve processing and ∆Tc statistics
- name: matplotlib
  role: Plotting molecular amount vs. temperature condensation curves
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202244850
    title: Revisiting equilibrium condensation and rocky planet compositions
  source_content_hash: sha256:88a1125903881c2f3a0e1c8d60ed6e63056a21e20e77e5ac62eba2a638f491e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AninaTimmermann/ECCOplanets
  tools:
  - ECCOplanets
  - Python
  - matplotlib
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# gibbs-free-energy-minimisation-condensation-simulation

## Summary

Run and analyse a T-p-dependent equilibrium condensation simulation (ECCOplanets) based on Gibbs free energy minimisation to derive condensation temperatures (Tc) of planet-building elements and quantify their sensitivity to disk pressure, metallicity, and element ratios such as C/O.

## When to use

Use when you need 50% condensation temperatures (Tc) or ∆Tc of major rock-forming elements (e.g. Mg, Si, Fe, Al, Ca, Na, Ni) under a given stellar abundance pattern, or when quantifying how Tc shifts with disk pressure (e.g. 10^-6–0.1 bar) or metallicity.

## When NOT to use

- You need gas-phase-only equilibrium chemistry — the paper notes the TEA code is 'limited to gas-phase simulations', so a gas-phase code is more appropriate if no condensation is involved
- Your abundance data is very incomplete (e.g. Al, Ca, Mg, Si, or O not reported) — the README cautions to consider 'the sensibility of running simulations on very incomplete abundance data sets'
- You need solid-solution mineralogy — 'We only consider pure minerals at the moment', so mixed-crystal equilibria require another tool

## Inputs

- Elemental abundance pattern (e.g. solar abundances or F/G/K-star data from the Brewer+2016 database)
- Thermochemical database (.pkl files read via data.py, containing tabulated and Shomate data)
- Simulation configuration parameters (temperature range and resolution, molecule selection, pressure grid)

## Outputs

- Molecular amount vs. temperature condensation curves for each pressure point
- Per-element 50% condensation temperatures (Tc)
- Per-element and mean ∆Tc tables (e.g. ∆Tc ≈ +350 K for 10^-6–0.1 bar; mean 357 ± 57 K)
- Rocky planet bulk compositions and devolatilisation (via s3planet_composition.py)

## How to apply

Configure the simulation via simulation.py (following sample s1simulation.py), setting the temperature range and resolution, molecule selection, and the elemental abundance pattern (e.g. solar, or Brewer+2016 stellar data). Run the Gibbs free energy minimisation at logarithmically spaced pressures from 1e-6 to 1e-1 bar to obtain molecular amount vs. temperature curves. Then improve each result per s2analyses.py by cropping the gas phase, recomputing a section, removing numerical glitches, and smoothing the curve before extracting the 50% condensation temperature per element. Compute per-element ∆Tc relative to a 1e-6 bar baseline and the mean ∆Tc using pandas/numpy; validated values include mean ∆Tc ≈ +357 ± 57 K for the tested pressure range and ∆Tc ≈ −100 K for C/O increasing from 0.1 to 0.7. The model itself was validated to reproduce literature Tc within ±5 K for common planet-building elements.

## Related tools

- **ECCOplanets** (Core open-source Python code that performs the Gibbs free energy minimisation condensation simulation and provides the thermochemical and stellar abundance databases) — https://github.com/AninaTimmermann/ECCOplanets
- **Python** (Language in which the code is built; simulation.py holds the mathematical and thermochemical formulation, and analysis uses pandas, numpy, matplotlib)
- **pandas** (Data handling for simulation curves, Tc extraction, and ∆Tc result tables)
- **numpy** (Numerical computation for condensation curve processing and ∆Tc statistics)
- **matplotlib** (Plotting molecular amount vs. temperature condensation curves)

## Examples

```
Run the sample workflow: `python s1simulation.py` (configure T range, molecule selection, abundance pattern, and pressure grid 1e-6–1e-1 bar in simulation.py), then `python s2analyses.py` to load the simulation, improve curves, and extract per-element Tc.
```

## Evaluation signals

- Derived 50% Tc values for common planet-building elements must fall within ±5 K of literature condensation temperatures
- Mean ∆Tc for the 10^-6–0.1 bar pressure sweep must reproduce 357 ± 57 K (∆Tc ≈ +350 K)
- C/O increase from 0.1 to 0.7 must yield ∆Tc ≈ −100 K; -0.3 ≤ [M/H] ≤ 0.4 should give ∆Tc ≈ +100 K
- Condensation curves should be free of numerical glitches and smoothing artefacts after applying the s2analyses.py improvement steps (cropping the gas phase, recomputing a section)
- Simulation outputs (.pkl el_amounts files) must load via data.py and expose basic simulation parameters via the simulation class

## Limitations

- Only pure minerals are considered at the moment; solid solutions are planned for a later release
- First release only includes F, G, and K star abundance data from Brewer+2016; other stellar types require added data and uncertainty consideration
- Applying element volatility determined from Solar System parameters to general planet-formation theories is limited — Tc depends on pressure and metallicity
- The code is a simplified starting point for approximate planetary compositions, not high-fidelity thermochemical modelling
- Condensation simulation results can contain numerical glitches requiring recomputation of sections and curve smoothing

## Evidence

- [abstract] Our T-p-dependent chemical equilibrium model is based on a Gibbs free energy minimisation.: "Our T-p-dependent chemical equilibrium model is based on a Gibbs free energy minimisation."
- [abstract] Model validation against literature: "For common planet-building elements, we derive a Tc that is within ±5 K of literature values"
- [abstract] Pressure sensitivity finding: "The tested pressure range (10−6 − 0.1 bar) corresponds to ∆Tc ≈ +350 K, and for −0.3 ≤ [M/H] ≤ 0.4 we find ∆Tc ≈ +100 K"
- [methods] Simulation sample workflow: "In this sample we introduce the parameters of the condensation simulation, i.e. the temperature range and resolution, the molecule selection, and the elemental abundance pattern."
- [methods] Result improvement steps: "improving the simulation result by recomputing a section, removing numerical glitches, and smoothing the curve"
- [methods] Code dependencies: "The code is built in python and uses the common packages pandas, numpy and matplotlib."
- [readme] simulation.py role: "this is the backbone of our condensation simulation. It contains the mathematical and thermochemical formulation of our code and the parameters of the optimisation procedure."
- [readme] Pure minerals caveat: "We only consider pure minerals at the moment, but there are plans to add solid solutions at a later point."
