---
name: rocky-planet-bulk-composition-analysis
description: Use when you have equilibrium condensation simulation results (molecule
  amounts vs temperature) and host-star elemental abundance patterns, and need to
  derive condensation temperatures (Tc) and rocky planet bulk compositions/devolatilisation
  trends.
when_to_use_negative:
- 'When studying gas giants or volatile-dominated planets: the code models rocky planet
  compositions from condensation of planet-building elements and only considers pure
  minerals at the moment'
- 'When the stellar abundance data is very incomplete (e.g. Al, Ca, Mg, Si, or O not
  reported): running simulations on such data is of questionable sensibility per the
  README'
- 'When gas-phase chemistry alone suffices: the condensation simulation step is intended
  for solid-forming species, unlike gas-phase-only codes such as TEA'
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: ECCOplanets
  role: Provides the simulation class for improving condensation curves and extracting
    condensation temperatures, and the planet class for computing bulk composition
    and devolatilisation (samples s2analyses.py and s3planet_composition.py)
  repo: https://github.com/AninaTimmermann/ECCOplanets
- name: Python
  role: Runtime environment with pandas, numpy and matplotlib used to execute the
    ECCOplanets samples
- name: pandas
  role: Used by data.py to load the .pkl thermochemical and stellar abundance databases
    and by the simulation/planet classes for tabular analysis
- name: numpy
  role: Numerical backend for the condensation simulation and curve analysis
- name: matplotlib
  role: Plots of molecule amounts vs temperature and planet composition results via
    pltStyle.py
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1051/0004-6361/202244850
    title: Revisiting equilibrium condensation and rocky planet compositions
  source_content_hash: sha256:b8a3b3401bab0be7de7e4eb281155df4ae383162d0b0e87beefa78c97a4e0fd7
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
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# rocky-planet-bulk-composition-analysis

## Summary

Compute rocky planet bulk compositions and devolatilisation patterns from equilibrium condensation simulations using the ECCOplanets planet class, condensing the simulated molecule-vs-temperature curves into element condensation temperatures (50% condensation points) and composition-temperature relationships relative to each host star.

## When to use

Use when you have equilibrium condensation simulation results (molecule amounts vs temperature) and host-star elemental abundance patterns, and need to derive condensation temperatures (Tc) and rocky planet bulk compositions/devolatilisation trends.

## When NOT to use

- When studying gas giants or volatile-dominated planets: the code models rocky planet compositions from condensation of planet-building elements and only considers pure minerals at the moment
- When the stellar abundance data is very incomplete (e.g. Al, Ca, Mg, Si, or O not reported): running simulations on such data is of questionable sensibility per the README
- When gas-phase chemistry alone suffices: the condensation simulation step is intended for solid-forming species, unlike gas-phase-only codes such as TEA

## Inputs

- Equilibrium condensation simulation results from ECCOplanets (molecule amounts vs temperature curves)
- Host-star elemental abundance pattern from the ECCOplanets stellar abundance database (.pkl files read via data.py, e.g. Brewer+2016 F/G/K star data)
- Thermochemical database (.pkl) for molecule class exploration
- Simulation parameters: temperature range and resolution, molecule selection, elemental abundance pattern

## Outputs

- Condensation temperatures (Tc, 50% condensation point) of planet-building elements per star
- Rocky planet bulk composition as a function of temperature
- Devolatilisation of the planet composition relative to the host star
- Improved simulation curves (gas phase cropped, glitch sections recomputed, smoothed)

## How to apply

Run a condensation simulation with simulation.py using the star-specific elemental abundance pattern (e.g. from the Brewer+2016 database, S/N > 100, log g > 3.5 F/G/K subsample), specifying temperature range, resolution, and molecule selection as shown in s1simulation.py. Improve the raw simulation with the classes.py simulation class (s2analyses.py): crop the gas phase, recompute the affected section to remove numerical glitches, and smooth the curve. Extract the condensation temperature of each planet-building element as the 50% condensation point from the improved molecule-vs-temperature curves using the simulation class analysis functions. Then, with the planet class (following s3planet_composition.py), compute rocky planet bulk compositions as a function of temperature and devolatilisation relative to the host star. Validate by checking derived Tc values against literature within ±5 K and confirming trend reproduction (Tc rising with metallicity, ~-100 K shift for C/O increase from 0.1 to 0.7).

## Related tools

- **ECCOplanets** (Provides the simulation class for improving condensation curves and extracting condensation temperatures, and the planet class for computing bulk composition and devolatilisation (samples s2analyses.py and s3planet_composition.py)) — https://github.com/AninaTimmermann/ECCOplanets
- **Python** (Runtime environment with pandas, numpy and matplotlib used to execute the ECCOplanets samples)
- **pandas** (Used by data.py to load the .pkl thermochemical and stellar abundance databases and by the simulation/planet classes for tabular analysis)
- **numpy** (Numerical backend for the condensation simulation and curve analysis)
- **matplotlib** (Plots of molecule amounts vs temperature and planet composition results via pltStyle.py)

## Examples

```
python s3planet_composition.py  # run simulation, improve result (crop gas phase, recompute section, smooth), compute rocky planet composition and devolatilisation vs temperature
```

## Evaluation signals

- Derived condensation temperatures for common planet-building elements agree with literature values within ±5 K
- Computed Tc rises with metallicity over the range -0.3 ≤ [M/H] ≤ 0.4 (∆Tc ≈ +100 K) as found in the paper
- Increase in C/O from 0.1 to 0.7 reproduces the expected ∆Tc ≈ -100 K shift
- Improved simulation curves show no numerical glitches after recomputing the affected section and smoothing
- Rocky planet bulk compositions vary smoothly with temperature and devolatilisation is consistently reported relative to the host star pattern

## Limitations

- Only pure minerals are considered at the moment; solid solutions are planned for later releases
- The first release only includes abundance data of F, G, and K stars from the Brewer+2016 database
- Simulations on very incomplete abundance data sets (e.g. missing Al, Ca, Mg, Si, or O) may not be sensible
- Feeding-zone model choice affects planetary composition output; the three FZ models must be selected/adapted deliberately

## Evidence

- [methods] Condensation temperatures are 50% condensation points derived from molecule-vs-temperature curves: "computing the composition of rocky planets as a function of temperature and analysing these planets, with regard to the condensation temperature of elements, bulk composition and devolatilisation"
- [readme] Simulation improvement involves cropping gas phase, recomputing sections, and smoothing: "improving the simulation result by recomputing a section, removing numerical glitches, and smoothing the curve"
- [abstract] Tc accuracy target vs literature is ±5 K for planet-building elements: "For common planet-building elements, we derive a Tc that is within ±5 K of literature values"
- [abstract] C/O ratio sensitivity produces a -100 K shift in condensation temperatures: "An increase in C/O from 0.1 to 0.7 results in a decrease of ∆Tc ≈ −100 K."
- [readme] Planet class computes composition as function of temperature and devolatilisation: "the planet class: functions to analyse the simulated rocky planets (mostly shown in sample s3planet_composition.py"
