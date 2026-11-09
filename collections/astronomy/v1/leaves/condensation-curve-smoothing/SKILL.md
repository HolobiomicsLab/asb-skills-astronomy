---
name: condensation-curve-smoothing
description: Use when raw ECCOplanets equilibrium condensation output (element/molecule
  abundances vs temperature, 2000-300 K) shows numerical glitches or jagged curves
  that must be cleaned before extracting 50% condensation temperatures or ∆Tc trends.
when_to_use_negative:
- You need to change the scientific assumptions of the condensation calculation itself
  (gas-minimisation setup, molecule set, pressure) — that belongs in simulation.py,
  not post-processing
- You want to expand the thermochemical or stellar abundance databases (s4adding_data.py)
  rather than clean an existing simulation result
- The raw abundance curves are already smooth and the 50% condensation temperatures
  reproduce benchmark values — unnecessary cleaning could distort the curves
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3855
tools:
- name: ECCOplanets
  role: Provides the simulation class (classes.py) and sample s2analyses.py whose
    functions crop the gas phase, recompute sections, remove numerical glitches, and
    smooth curves
  repo: https://github.com/AninaTimmermann/ECCOplanets
- name: Python
  role: Language in which the cleaning/analysis functions of the simulation class
    are implemented and run
  repo: https://github.com/AninaTimmermann/ECCOplanets
- name: pandas
  role: Handles the .pkl databases and tabular simulation results during curve cleaning
    and condensation-temperature extraction
- name: matplotlib
  role: Plots molecule/element abundance vs temperature before and after smoothing
    to visually verify the cleaned curves
provenance:
  source_task_ids:
  - task_003
  - task_007
  source_papers:
  - doi: 10.1051/0004-6361/202244850
    title: Revisiting equilibrium condensation and rocky planet compositions
  source_content_hash: sha256:3391aeee57e75ce46d5887156de708e5b244b047c601cfd63c1ac333d7c8b04e
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3855
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# condensation-curve-smoothing

## Summary

Post-processing of ECCOplanets Gibbs-free-energy-minimisation condensation simulations to clean raw abundance-vs-temperature curves: cropping the gas phase, recomputing glitched sections, removing numerical glitches, and smoothing before extracting condensation temperatures.

## When to use

Use when raw ECCOplanets equilibrium condensation output (element/molecule abundances vs temperature, 2000-300 K) shows numerical glitches or jagged curves that must be cleaned before extracting 50% condensation temperatures or ∆Tc trends.

## When NOT to use

- You need to change the scientific assumptions of the condensation calculation itself (gas-minimisation setup, molecule set, pressure) — that belongs in simulation.py, not post-processing
- You want to expand the thermochemical or stellar abundance databases (s4adding_data.py) rather than clean an existing simulation result
- The raw abundance curves are already smooth and the 50% condensation temperatures reproduce benchmark values — unnecessary cleaning could distort the curves

## Inputs

- ECCOplanets simulation output written to the simulations folder (element and molecule abundances as a function of temperature)
- Simulation configuration from s1simulation.py (temperature range/resolution, molecule selection, pressure, elemental abundance pattern)
- ECCOplanets thermochemical and stellar abundance .pkl databases loaded via data.py

## Outputs

- Cleaned abundance-vs-temperature curves (glitch-free, smoothed, cropped gas phase)
- Validated 50% condensation temperatures per element (CSV table, e.g. condensation_temperatures_co_variation.csv)
- Simulation object ready for s3planet_composition.py bulk-composition and devolatilisation analysis

## How to apply

After running a condensation simulation via simulation.py (s1simulation.py, e.g. 2000 K down to 300 K at 1e-4 bar), load the result in s2analyses.py using the simulation class from classes.py. Improve the result in the sequence demonstrated in the samples: crop the gas phase, recompute a section of the temperature grid where the optimisation failed, remove numerical glitches, and smooth the abundance curves. This matters because raw output glitches propagate into 50% condensation temperature extraction (e.g. Al 1643 K, Ti 1575 K, Ca 1512 K, Mg 1331 K, Fe 1329 K, Si 1305 K at 1e-4 bar) and into ∆Tc comparisons such as the ≈ −100 K shift when C/O rises from 0.1 to 0.7. The same sequence is also embedded in s3planet_composition.py before computing planet bulk compositions and devolatilisation. Verify by re-plotting the cleaned abundance curves and confirming the derived condensation temperatures agree with literature values within the paper's ±5 K tolerance.

## Related tools

- **ECCOplanets** (Provides the simulation class (classes.py) and sample s2analyses.py whose functions crop the gas phase, recompute sections, remove numerical glitches, and smooth curves) — https://github.com/AninaTimmermann/ECCOplanets
- **Python** (Language in which the cleaning/analysis functions of the simulation class are implemented and run) — https://github.com/AninaTimmermann/ECCOplanets
- **pandas** (Handles the .pkl databases and tabular simulation results during curve cleaning and condensation-temperature extraction)
- **matplotlib** (Plots molecule/element abundance vs temperature before and after smoothing to visually verify the cleaned curves)

## Examples

```
sim = load_simulation('solar_1e-4bar'); sim.crop_gas_phase(); sim.recompute_section(); sim.remove_glitches(); sim.smooth_curves(); sim.condensation_temperature('Fe')
```

## Evaluation signals

- Derived 50% condensation temperatures for Al, Ti, Ca, Mg, Fe, Si match the reported values (1643, 1575, 1512, 1331, 1329, 1305 K at 1e-4 bar) with average discrepancy < 5 K and largest deviation < 15 K
- Output CSV contains all six major rock-forming elements
- Cleaned abundance curves are smooth and free of numerical glitches when re-plotted (line plots of amount vs temperature)
- ∆Tc over the C/O 0.1 → 0.7 sweep reproduces the reported ≈ −100 K decrease after cleaning
- Gas-phase cropping does not change condensation temperatures beyond the stated agreement tolerance

## Limitations

- Cleaning is a numerical post-process only; it cannot fix wrong scientific assumptions in the Gibbs free energy minimisation (those must be changed in simulation.py)
- The smoothing/cropping sequence is demonstrated in sample files and tuned to the default temperature range/resolution and molecule selection; other settings may need re-tuning
- Curves from simulations on very incomplete abundance data (e.g. missing Al, Ca, Mg, Si, or O) may be unreliable regardless of cleaning

## Evidence

- [readme] improving the simulation result by recomputing a section, removing numerical glitches, and smoothing the curve: "improving the simulation result by recomputing a section, removing numerical glitches, and smoothing the curve"
- [readme] cropping the gas phase and recomputing a section: "This sample takes you from running a simulation, over improving the result (cropping the gas phase and recomputing a section)"
- [methods] extract 50% condensation temperature as half of total abundance condensed: "In this sample we introduce the parameters of the condensation simulation, i.e. the temperature range and resolution, the molecule selection, and the elemental abundance pattern."
- [abstract] validation within ±5 K of literature: "Our model reproduces the external results well in all tests. For common planet-building elements, we derive a Tc that is within ±5 K of literature values"
