---
name: elemental-ratio-variation-analysis
description: Use when you need to isolate the effect of a single stellar/disk abundance
  ratio (e.g. C/O) on element condensation temperatures in an equilibrium condensation
  simulation, keeping temperature range, pressure, molecule selection and all other
  element ratios fixed.
when_to_use_negative:
- The stellar abundance data set is very incomplete (e.g. Al, Ca, Mg, Si or O data
  not reported) — the README warns to consider the sensibility of running simulations
  on such data.
- You need solid-solution condensates rather than pure minerals — the code 'only consider[s]
  pure minerals at the moment'.
- The goal is sensitivity to pressure or metallicity rather than to one elemental
  ratio — those axes are varied separately (pressure 10^-6–0.1 bar, [M/H] −0.3 to
  0.4).
edam_operation: http://edamontology.org/operation_0242
edam_topics:
- http://edamontology.org/topic_3335
tools:
- name: ECCOplanets
  role: Gibbs free energy minimisation condensation code (simulation.py, s1simulation.py,
    s2analyses.py) used to run each C/O setting and extract Tc/∆Tc
  repo: https://github.com/AninaTimmermann/ECCOplanets
- name: Python
  role: Language in which ECCOplanets is built, using pandas, numpy and matplotlib
    for table/plot outputs
- name: pandas
  role: Reads the .pkl databases and assembles the condensation_temperatures_co_variation.csv
    table
- name: matplotlib
  role: Plots the Tc and ∆Tc trends versus C/O
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1051/0004-6361/202244850
    title: Revisiting equilibrium condensation and rocky planet compositions
  source_content_hash: sha256:3fcad42b12f42cc35c35b5a795ac71c19398e66aec5a54a4b55575372f2ded81
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
  edam_operation: http://edamontology.org/operation_0242
  edam_topics:
  - http://edamontology.org/topic_3335
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# elemental-ratio-variation-analysis

## Summary

Quantifies the sensitivity of Gibbs free energy minimisation condensation temperatures to disk abundance patterns by varying one key element ratio at a time (C/O from 0.1 to 0.7), extracting condensation temperatures (Tc) of common planet-building elements, and computing ∆Tc relative to a C/O = 0.1 baseline. Uses the ECCOplanets Python code to show that C/O dominates with ∆Tc ≈ −100 K while other ratios are less influential.

## When to use

Use when you need to isolate the effect of a single stellar/disk abundance ratio (e.g. C/O) on element condensation temperatures in an equilibrium condensation simulation, keeping temperature range, pressure, molecule selection and all other element ratios fixed.

## When NOT to use

- The stellar abundance data set is very incomplete (e.g. Al, Ca, Mg, Si or O data not reported) — the README warns to consider the sensibility of running simulations on such data.
- You need solid-solution condensates rather than pure minerals — the code 'only consider[s] pure minerals at the moment'.
- The goal is sensitivity to pressure or metallicity rather than to one elemental ratio — those axes are varied separately (pressure 10^-6–0.1 bar, [M/H] −0.3 to 0.4).

## Inputs

- Thermochemical database (.pkl files in the ECCOplanets data folder, loaded via data.py)
- Stellar abundance database (.pkl, F/G/K stars from Brewer+2016)
- Elemental abundance patterns spanning C/O = 0.1–0.7 (C and O abundances adapted via the parabolic C/O–Σ/O correlation, all other ratios fixed)
- Fixed simulation parameters: temperature range and resolution, molecule selection, pressure

## Outputs

- Condensation temperatures (Tc) of common planet-building elements for each C/O setting
- ∆Tc values relative to the C/O = 0.1 baseline per element
- Tabulated Tc and ∆Tc versus C/O saved as condensation_temperatures_co_variation.csv
- Matplotlib plot of the Tc/∆Tc versus C/O trend

## How to apply

Load the thermochemical databases and stellar abundance data (.pkl files read by data.py), then generate abundance patterns spanning C/O = 0.1 to C/O = 0.7 by adapting the carbon and oxygen abundances following the parabolic correlation of C/O with Σ/O observed in the Brewer+2016 stellar sample, holding all other element ratios fixed. For each C/O value run the Gibbs free energy minimisation condensation simulation (s1simulation.py) with unchanged temperature range, resolution, molecule selection and pressure, so ∆Tc is attributable purely to the varied ratio. Load each result with s2analyses.py and analyse condensation of molecules and elements, improving results by cropping the gas phase, recomputing glitched sections, removing numerical glitches and smoothing the curves. Extract the condensation temperatures of the common planet-building elements for every C/O setting, compute ∆Tc against the C/O = 0.1 baseline, tabulate and plot Tc and ∆Tc versus C/O, and save the table as condensation_temperatures_co_variation.csv. Validation criterion: the simulated ∆Tc for the C/O increase from 0.1 to 0.7 should reproduce the reported decrease of approximately −100 K.

## Related tools

- **ECCOplanets** (Gibbs free energy minimisation condensation code (simulation.py, s1simulation.py, s2analyses.py) used to run each C/O setting and extract Tc/∆Tc) — https://github.com/AninaTimmermann/ECCOplanets
- **Python** (Language in which ECCOplanets is built, using pandas, numpy and matplotlib for table/plot outputs)
- **pandas** (Reads the .pkl databases and assembles the condensation_temperatures_co_variation.csv table)
- **matplotlib** (Plots the Tc and ∆Tc trends versus C/O)

## Examples

```
python data.py; python samples/s1simulation.py  # run for C/O = 0.1...0.7 abundance patterns; python samples/s2analyses.py  # extract Tc, smooth, tabulate
```

## Evaluation signals

- Simulated ∆Tc for the C/O increase from 0.1 to 0.7 matches the reported decrease of approximately −100 K
- Variations in other element ratios produce smaller ∆Tc than the C/O variation
- Baseline Tc values for common planet-building elements fall within ±5 K of literature values, as the model was validated externally
- Simulation parameters (temperature range, resolution, molecule selection, pressure) are unchanged across all C/O runs
- Output table condensation_temperatures_co_variation.csv contains Tc and ∆Tc for every C/O setting

## Limitations

- Only pure minerals are considered at present; solid solutions are planned but not implemented
- Stellar abundance data limited to F, G, and K stars from the Brewer+2016 database in the first release
- Running simulations on very incomplete abundance data sets (e.g. missing Al, Ca, Mg, Si, or O) may be insensible
- Simulation results can contain numerical glitches requiring cropping the gas phase, recomputing a section, and smoothing the curve

## Evidence

- [abstract] An increase in C/O from 0.1 to 0.7 results in a decrease of ∆Tc ≈ −100 K. Other element ratios are less influential.: "An increase in C/O from 0.1 to 0.7 results in a decrease of ∆Tc ≈ −100 K. Other element ratios are less influential."
- [abstract] To analyse the influence of the abundance pattern, key element ratios were varied, and the results were validated using solar neighbourhood stars.: "To analyse the influence of the abundance pattern, key element ratios were varied, and the results were validated using solar neighbourhood stars."
- [readme] improving the simulation result by recomputing a section, removing numerical glitches, and smoothing the curve: "improving the simulation result by recomputing a section, removing numerical glitches, and smoothing the curve"
- [readme] This first release only includes abundance data of F, G, and K stars from the Brewer+2016 database.: "This first release only includes abundance data of F, G, and K stars from the Brewer+2016 database."
- [abstract] For common planet-building elements, we derive a Tc that is within ±5 K of literature values: "For common planet-building elements, we derive a Tc that is within ±5 K of literature values"
