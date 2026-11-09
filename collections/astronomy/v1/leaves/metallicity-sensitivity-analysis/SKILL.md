---
name: metallicity-sensitivity-analysis
description: Use when you need to test the sensitivity of equilibrium condensation
  temperatures to metallicity offsets, e.g. to assess how stellar [M/H] propagates
  into rocky-planet building-block volatility while holding relative element ratios
  fixed.
when_to_use_negative:
- The question concerns variation of individual element ratios (e.g. C/O from 0.1
  to 0.7, which gives ∆Tc ≈ −100 K) rather than an overall metallicity offset — use
  the element-ratio sensitivity workflow instead
- The question concerns disk pressure variation (10^-6–0.1 bar, ∆Tc ≈ +350 K) rather
  than metallicity
- The goal is computing bulk rocky-planet compositions or devolatilisation via feeding-zone
  models (s3planet_composition.py) rather than condensation-temperature sensitivity
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3895
tools:
- name: ECCOplanets
  role: Open-source Python code providing the Gibbs free energy minimisation condensation
    simulation, thermochemical databases, and analysis samples (s1simulation.py, s2analyses.py)
    used to run the metallicity sensitivity study
  repo: https://github.com/AninaTimmermann/ECCOplanets
- name: Python
  role: Implementation language of the ECCOplanets simulation and analysis pipeline
- name: pandas
  role: Reads the .pkl thermochemical and stellar abundance databases into DataFrames
    via data.py
- name: numpy
  role: Numerical backend for the equilibrium optimisation and abundance-pattern scaling
- name: matplotlib
  role: Plotting of condensation curves and Tc vs. [M/H] sensitivity relations (styled
    via pltStyle.py)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202244850
    title: Revisiting equilibrium condensation and rocky planet compositions
  source_content_hash: sha256:0b569f31617144610dbb078b1cdeaa75c6210fd2380d74b1122f8ceb12fdc1fa
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3895
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# metallicity-sensitivity-analysis

## Summary

A workflow for quantifying how the 50% condensation temperatures (Tc) of planet-building elements respond to systematic offsets in overall disk metallicity, using the ECCOplanets Gibbs free energy minimisation code. It reproduces the paper's finding that scaling the solar abundance pattern over -0.3 ≤ [M/H] ≤ 0.4 raises Tc by ≈ +100 K for most elements.

## When to use

Use when you need to test the sensitivity of equilibrium condensation temperatures to metallicity offsets, e.g. to assess how stellar [M/H] propagates into rocky-planet building-block volatility while holding relative element ratios fixed.

## When NOT to use

- The question concerns variation of individual element ratios (e.g. C/O from 0.1 to 0.7, which gives ∆Tc ≈ −100 K) rather than an overall metallicity offset — use the element-ratio sensitivity workflow instead
- The question concerns disk pressure variation (10^-6–0.1 bar, ∆Tc ≈ +350 K) rather than metallicity
- The goal is computing bulk rocky-planet compositions or devolatilisation via feeding-zone models (s3planet_composition.py) rather than condensation-temperature sensitivity

## Inputs

- Thermochemical data database (.pkl files in the ECCOplanets data folder)
- Stellar abundance database (.pkl files; Brewer+2016 F, G, and K star abundances)
- Solar elemental abundance pattern with molecule selection and simulation parameters (temperature range and resolution)
- Metallicity offset grid spanning -0.3 to +0.4 dex applied to the solar abundance pattern

## Outputs

- Per-element 50% condensation temperatures Tc([M/H]) for each metallicity offset
- ∆Tc values relative to the solar-composition run (expected ≈ +100 K over -0.3 ≤ [M/H] ≤ 0.4 for most elements)
- Condensation curves (molecule/element amounts vs. temperature) after gas-phase cropping, glitch removal, and smoothing

## How to apply

Load the thermochemical and stellar abundance .pkl databases via data.py, then instantiate the simulation class following sample s1simulation.py, fixing the temperature range, resolution, molecule selection, and the solar elemental abundance pattern. Generate synthetic abundance patterns by scaling the solar pattern to metallicity offsets spanning -0.3 to +0.4 dex while keeping relative element ratios unchanged, and run the Gibbs free energy minimisation condensation simulation (simulation.py) for each [M/H]. Post-process each run with the s2analyses.py operations — cropping the gas phase, recomputing an affected section, removing numerical glitches, and smoothing the curve — before extracting the 50% condensation temperature of each planet-building element and computing ∆Tc relative to the solar-composition run. Validate by confirming the average ∆Tc over the -0.3 ≤ [M/H] ≤ 0.4 range is ≈ +100 K for most elements, and that baseline solar Tc values match literature values within ±5 K.

## Related tools

- **ECCOplanets** (Open-source Python code providing the Gibbs free energy minimisation condensation simulation, thermochemical databases, and analysis samples (s1simulation.py, s2analyses.py) used to run the metallicity sensitivity study) — https://github.com/AninaTimmermann/ECCOplanets
- **Python** (Implementation language of the ECCOplanets simulation and analysis pipeline)
- **pandas** (Reads the .pkl thermochemical and stellar abundance databases into DataFrames via data.py)
- **numpy** (Numerical backend for the equilibrium optimisation and abundance-pattern scaling)
- **matplotlib** (Plotting of condensation curves and Tc vs. [M/H] sensitivity relations (styled via pltStyle.py))

## Examples

```
from data import *; from simulation import Simulation; sim = Simulation(T_range, resolution, molecules, solar_abundance_scaled_to_MH); sim.run()
```

## Evaluation signals

- Average ∆Tc over the -0.3 ≤ [M/H] ≤ 0.4 range is ≈ +100 K for most planet-building elements, matching the paper's reported value
- Baseline solar-composition 50% condensation temperatures agree with literature values within ±5 K
- Relative element ratios are unchanged when scaling the abundance pattern to each metallicity offset (only the overall [M/H] shifts)
- Post-processed condensation curves show no numerical glitches and are smoothed per the s2analyses.py improvement steps (cropping the gas phase, recomputing a section)
- Metallicity grid covers at least the endpoints -0.3 and +0.4 dex so the full reported ∆Tc range is bracketed

## Limitations

- The condensation simulation is limited to the most common chemical species, so Tc values are approximations; ECCOplanets currently considers only pure minerals (solid solutions planned but not included)
- The bundled stellar abundance database only covers F, G, and K stars from the Brewer+2016 database; simulations on very incomplete abundance sets (missing Al, Ca, Mg, Si, or O) may not be sensible
- The +100 K ∆Tc result holds 'for most elements'; individual elements may deviate from the average trend

## Evidence

- [abstract] For most elements, it rises with p and metallicity. The tested pressure range (10−6 − 0.1 bar) corresponds to ∆Tc ≈ +350 K, and for −0.3 ≤ [M/H] ≤ 0.4 we find ∆Tc ≈ +100 K: "For most elements, it rises with p and metallicity. The tested pressure range (10−6 − 0.1 bar) corresponds to ∆Tc ≈ +350 K, and for −0.3 ≤ [M/H] ≤ 0.4 we find ∆Tc ≈ +100 K"
- [abstract] To analyse the influence of the abundance pattern, key element ratios were varied, and the results were validated using solar neighbourhood stars.: "To analyse the influence of the abundance pattern, key element ratios were varied, and the results were validated using solar neighbourhood stars."
- [methods] In this sample we introduce the parameters of the condensation simulation, i.e. the temperature range and resolution, the molecule selection, and the elemental abundance pattern.: "In this sample we introduce the parameters of the condensation simulation, i.e. the temperature range and resolution, the molecule selection, and the elemental abundance pattern."
- [methods] improving the simulation result by recomputing a section, removing numerical glitches, and smoothing the curve: "improving the simulation result by recomputing a section, removing numerical glitches, and smoothing the curve"
- [readme] data.py this code only reads the .pkl files containing all relevant data into python: "this code only reads the .pkl files containing all relevant data into python"
- [abstract] Our model reproduces the external results well in all tests. For common planet-building elements, we derive a Tc that is within ±5 K of literature values: "For common planet-building elements, we derive a Tc that is within ±5 K of literature values"
