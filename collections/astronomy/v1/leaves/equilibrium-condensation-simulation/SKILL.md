---
name: equilibrium-condensation-simulation
description: Use when you need condensation sequences or 50% condensation temperatures
  (Tc) of elements/molecules from a stellar abundance pattern, or bulk compositions
  of rocky planets accreted from a feeding zone, e.g. testing metallicity or C/O sensitivity
  of Tc.
when_to_use_negative:
- 'You need gas-phase-only chemistry: TEA is noted as limited to gas-phase simulations,
  whereas this code includes condensation — but if no condensed species matter, a
  gas-phase code may suffice'
- The stellar abundance data set is very incomplete (e.g. Al, Ca, Mg, Si, or O not
  reported) — the README warns to think about the sensibility of running simulations
  on such data
- You need solid solutions rather than pure minerals — the code only considers pure
  minerals at the moment
edam_operation: http://edamontology.org/operation_0250
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: ECCOplanets
  role: Open-source Python code performing the Gibbs free energy minimisation condensation
    simulation and providing the molecule, simulation, and planet analysis classes
  repo: https://github.com/AninaTimmermann/ECCOplanets
- name: Python
  role: Language in which ECCOplanets is built, together with pandas, numpy, and matplotlib
- name: pandas
  role: Reads the .pkl thermochemical and stellar abundance databases via data.py
    and handles simulation DataFrames
- name: numpy
  role: Numerical backend for the equilibrium computation and post-processing
- name: matplotlib
  role: Line plots of molecule amounts vs temperature and condensation curve figures
    (styling in pltStyle.py)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202244850
    title: Revisiting equilibrium condensation and rocky planet compositions
  source_content_hash: sha256:1c899337e8a8e854bfcfd7fa5051cc7c739d94f9314b426444f9b6ea8ce6c2bc
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
  edam_operation: http://edamontology.org/operation_0250
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# equilibrium-condensation-simulation

## Summary

Runs a T-p-dependent chemical equilibrium condensation simulation via Gibbs free energy minimisation using the ECCOplanets Python code, to derive condensation temperatures of planet-building elements and rocky planet bulk compositions from a given stellar elemental abundance pattern.

## When to use

Use when you need condensation sequences or 50% condensation temperatures (Tc) of elements/molecules from a stellar abundance pattern, or bulk compositions of rocky planets accreted from a feeding zone, e.g. testing metallicity or C/O sensitivity of Tc.

## When NOT to use

- You need gas-phase-only chemistry: TEA is noted as limited to gas-phase simulations, whereas this code includes condensation — but if no condensed species matter, a gas-phase code may suffice
- The stellar abundance data set is very incomplete (e.g. Al, Ca, Mg, Si, or O not reported) — the README warns to think about the sensibility of running simulations on such data
- You need solid solutions rather than pure minerals — the code only considers pure minerals at the moment

## Inputs

- Thermochemical database (.pkl files in the data folder, tabulated and fitted Shomate data)
- Stellar abundance database (.pkl; first release includes F, G, and K stars from Brewer+2016)
- Simulation parameters: temperature range and resolution, molecule selection, elemental abundance pattern (from s1simulation.py)

## Outputs

- Condensation curves (amount of molecules/elements as a function of temperature)
- 50% condensation temperatures (Tc) of planet-building elements
- Rocky planet bulk compositions as a function of temperature, with devolatilisation relative to the star (el_amounts files)

## How to apply

Load the thermochemical and stellar abundance .pkl databases with data.py, then instantiate the simulation class per sample s1simulation.py, fixing the temperature range and resolution, the molecule selection, and the elemental abundance pattern (e.g. solar, or scaled to a target [M/H] or C/O). Run the Gibbs free energy minimisation implemented in simulation.py, which is the mathematical and thermochemical backbone of the code and contains the optimisation parameters. Post-process with s2analyses.py: crop the gas phase, recompute a section, remove numerical glitches, and smooth the curve to obtain a clean condensation curve. Extract the 50% condensation temperature of each element or compute rocky planet composition with s3planet_composition.py. For new input species or stars, extend the databases via s4adding_data.py, noting that simulations on very incomplete abundance data sets (e.g. missing Al, Ca, Mg, Si, or O) may be nonsensical.

## Related tools

- **ECCOplanets** (Open-source Python code performing the Gibbs free energy minimisation condensation simulation and providing the molecule, simulation, and planet analysis classes) — https://github.com/AninaTimmermann/ECCOplanets
- **Python** (Language in which ECCOplanets is built, together with pandas, numpy, and matplotlib)
- **pandas** (Reads the .pkl thermochemical and stellar abundance databases via data.py and handles simulation DataFrames)
- **numpy** (Numerical backend for the equilibrium computation and post-processing)
- **matplotlib** (Line plots of molecule amounts vs temperature and condensation curve figures (styling in pltStyle.py))

## Examples

```
import data; from classes import simulation; sim = simulation('solar', Trange=[2000, 300], res=10); sim.run_condensation(); sim.Tc(elements=['Mg','Si','Fe'])
```

## Evaluation signals

- Derived condensation temperatures for common planet-building elements should fall within ±5 K of literature values (the paper's validation test)
- Baseline solar-composition condensation temperatures should reproduce the paper's Solar System Tc values
- Scaling metallicity over -0.3 ≤ [M/H] ≤ 0.4 should yield ∆Tc ≈ +100 K for most elements; pressure variation 10^-6–0.1 bar should give ∆Tc ≈ +350 K
- Increasing C/O from 0.1 to 0.7 should produce ∆Tc ≈ −100 K, while other element ratios are less influential
- Post-processed curves must be free of numerical glitches after the s2analyses.py cleaning steps (cropping, recomputation, smoothing)

## Limitations

- Only pure minerals are considered at the moment; solid solutions are planned for a later release
- First release includes abundance data only for F, G, and K stars from the Brewer+2016 database
- The code is a simplified starting point for approximate planetary compositions; changing scientific assumptions requires editing simulation.py
- Simulations on very incomplete abundance data sets (missing Al, Ca, Mg, Si, or O) may not be sensible
- Condensation temperatures are sensitive to disk pressure and metallicity (up to ∆Tc ≈ +350 K over 10^-6–0.1 bar), so applying Solar System-derived volatility to general planet formation theories has limitations

## Evidence

- [abstract] T-p-dependent Gibbs minimisation basis: "Our T-p-dependent chemical equilibrium model is based on a Gibbs free energy minimisation."
- [abstract] ±5 K validation against literature: "For common planet-building elements, we derive a Tc that is within ±5 K of literature values"
- [abstract] metallicity and pressure sensitivity: "The tested pressure range (10−6 − 0.1 bar) corresponds to ∆Tc ≈ +350 K, and for −0.3 ≤ [M/H] ≤ 0.4 we find ∆Tc ≈ +100 K"
- [readme] simulation parameters in s1simulation.py: "In this sample we introduce the parameters of the condensation simulation, i.e. the temperature range and resolution, 
the molecule selection, and the elemental abundance pattern."
- [readme] post-processing steps in s2analyses.py: "improving the simulation result by recomputing a section, removing numerical glitches, and smoothing the curve"
- [readme] simulation.py is the scientific backbone: "this is the backbone of our condensation simulation. It contains the mathematical and thermochemical formulation of our code and the parameters of the
  optimisation procedure. If you want to change"
- [readme] pure minerals only limitation: "We only consider pure minerals at the moment, but there are plans to add solid solutions at a later point."
- [readme] incomplete abundance data warning: "Also you might need to think about the sensibility of running simulations on very incomplete abundance
 data sets (e.g. if Al, Ca, Mg, Si, or O data is not reported)."
