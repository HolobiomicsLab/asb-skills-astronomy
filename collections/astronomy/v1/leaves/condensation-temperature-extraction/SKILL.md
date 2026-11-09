---
name: condensation-temperature-extraction
description: Use when you have run (or need to run) an ECCOplanets equilibrium condensation
  simulation and must convert element/molecule abundance-vs-temperature curves into
  per-element 50% condensation temperatures for validation or composition analysis.
when_to_use_negative:
- The goal is planet bulk composition or devolatilisation relative to the host star
  (s3planet_composition.py / planet class), not per-element condensation temperature
  extraction.
- The simulation is gas-phase only (e.g. the TEA code mentioned in the paper) — no
  condensation occurs, so no 50% condensation point exists.
- The stellar abundance pattern is too incomplete for condensation chemistry (e.g.
  Al, Ca, Mg, Si, or O data not reported), as the README warns against running simulations
  on such data.
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: ECCOplanets
  role: 'Equilibrium condensation code: runs the Gibbs free energy minimisation simulation
    and provides the simulation class analysis functions used to extract 50% condensation
    temperatures'
  repo: https://github.com/AninaTimmermann/ECCOplanets
- name: Python
  role: Implementation language; the code is built in python and uses pandas, numpy
    and matplotlib for simulation outputs and curve handling
  repo: https://github.com/AninaTimmermann/ECCOplanets
- name: classes.py (simulation class)
  role: 'Contains the simulation class functions for analysing condensation: cropping
    the gas phase, recomputing a section, removing numerical glitches, smoothing the
    curve, and extracting 50% condensation temperatures'
  repo: https://github.com/AninaTimmermann/ECCOplanets
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_008
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202244850
    title: Revisiting equilibrium condensation and rocky planet compositions
  source_content_hash: sha256:05a83b46442d3253283dd57b3035d2cc2effc6db4dda634b26e2995561c0d4a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AninaTimmermann/ECCOplanets
  tools:
  - ECCOplanets
  - Python
  - classes.py (simulation class)
  - matplotlib
  - numpy
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# condensation-temperature-extraction

## Summary

Extracts 50% condensation temperatures (Tc) of rock-forming elements from ECCOplanets Gibbs-free-energy-minimisation condensation simulations by post-processing molecule-vs-temperature abundance curves, and validates them against literature benchmark values within ±5 K.

## When to use

Use when you have run (or need to run) an ECCOplanets equilibrium condensation simulation and must convert element/molecule abundance-vs-temperature curves into per-element 50% condensation temperatures for validation or composition analysis.

## When NOT to use

- The goal is planet bulk composition or devolatilisation relative to the host star (s3planet_composition.py / planet class), not per-element condensation temperature extraction.
- The simulation is gas-phase only (e.g. the TEA code mentioned in the paper) — no condensation occurs, so no 50% condensation point exists.
- The stellar abundance pattern is too incomplete for condensation chemistry (e.g. Al, Ca, Mg, Si, or O data not reported), as the README warns against running simulations on such data.

## Inputs

- ECCOplanets thermochemical database (.pkl files read via data.py)
- Stellar abundance pattern database (.pkl, e.g. Lodders2003a solar abundances or Brewer+2016 F/G/K star abundances)
- Simulation configuration: temperature range/resolution, total pressure, molecule selection, elemental abundance pattern (s1simulation.py parameters)
- Simulation output: element and molecule amounts as a function of temperature written to the simulations folder

## Outputs

- Per-element 50% condensation temperatures (Tc) for rock-forming elements (e.g. Al, Ti, Ca, Mg, Fe, Si, Na, Ni)
- Improved condensation curves: molecule/element abundance vs. temperature after cropping, section recomputation, glitch removal, and smoothing
- CSV/table of condensation temperatures containing all analysed elements, usable for comparison against literature values

## How to apply

Configure a simulation in s1simulation.py with a temperature range (paper default 2000 K down to 300 K), a total pressure (1e-4 bar for Solar-System reference values), a molecule selection limited to the most common chemical species, and the target elemental abundance pattern (e.g. Lodders2003a solar). Run the Gibbs free energy minimisation in simulation.py, which writes element and molecule abundances as a function of temperature to the simulations folder. Then load the run per s2analyses.py and improve it with the simulation class in classes.py: crop the gas phase, recompute the affected section (which also removes numerical glitches), and smooth the abundance curves. Finally extract the 50% condensation temperature of each planet-building element as the temperature at which half of its total abundance has condensed, using the simulation class analysis functions. Validate the extracted Tc values against literature benchmarks — the paper's solar-pattern run yields Al 1643 K, Ti 1575 K, Ca 1512 K, Mg 1331 K, Fe 1329 K, Si 1305 K at p = 1e-4 bar, with average discrepancy below 5 K and largest deviation below 15 K.

## Related tools

- **ECCOplanets** (Equilibrium condensation code: runs the Gibbs free energy minimisation simulation and provides the simulation class analysis functions used to extract 50% condensation temperatures) — https://github.com/AninaTimmermann/ECCOplanets
- **Python** (Implementation language; the code is built in python and uses pandas, numpy and matplotlib for simulation outputs and curve handling) — https://github.com/AninaTimmermann/ECCOplanets
- **classes.py (simulation class)** (Contains the simulation class functions for analysing condensation: cropping the gas phase, recomputing a section, removing numerical glitches, smoothing the curve, and extracting 50% condensation temperatures) — https://github.com/AninaTimmermann/ECCOplanets

## Examples

```
python samples/s1simulation.py; python samples/s2analyses.py  # run condensation simulation, then analyse and extract 50% Tc
```

## Evaluation signals

- Solar-pattern Tc values at p = 1e-4 bar match the reported Al 1643 K, Ti 1575 K, Ca 1512 K, Ca/Mg/Fe/Si benchmarks (Mg 1331 K, Fe 1329 K, Si 1305 K); average discrepancy from literature mean condensation temperatures below 5 K, largest deviation below 15 K.
- The 50% condensation temperature is extracted as the temperature at which exactly half of the element's total abundance has condensed on the (smoothed, glitch-free) abundance-vs-temperature curve.
- The output CSV/table contains all six major rock-forming elements (Al, Ti, Ca, Mg, Fe, Si).
- Post-processing pipeline completed: gas phase cropped, affected section recomputed, numerical glitches removed, and curves smoothed before extraction.
- Sensitivity checks reproduce paper trends: mean ∆Tc ≈ +350 K over 1e-6–0.1 bar pressure variation and ∆Tc ≈ +100 K over −0.3 ≤ [M/H] ≤ 0.4 metallicity variation.

## Limitations

- Condensation temperatures are insensitive to details of the condensation algorithm (discrepancies <15 K), so the code is a simplified alternative to HSC Chemistry / GGchem rather than a high-fidelity replacement.
- The code only considers pure minerals at the moment; solid solutions are planned but not yet included, which affects condensation behaviour.
- Results depend on the molecule selection limited to the most common chemical species, and numerical glitches in raw simulation output require post-processing (cropping, recomputation, smoothing) before extraction.
- Simulations on very incomplete stellar abundance data sets (missing Al, Ca, Mg, Si, or O) may be insensible, per the README.

## Evidence

- [abstract] Benchmark solar Tc values and ±5 K agreement: "Our model reproduces the external results well in all tests. For common planet-building elements, we derive a Tc that is within ±5 K of literature values"
- [methods] Post-processing filter steps: "improving the simulation result by recomputing a section, removing numerical glitches, and smoothing the curve"
- [readme] Simulation parameters introduced in s1simulation.py: "In this sample we introduce the parameters of the condensation simulation, i.e. the temperature range and resolution, the molecule selection, and the elemental abundance pattern."
- [readme] Simulation class analysis scope: "the simulation class: functions to analyse the condensation simulation (mostly shown in sample s2analyses.py)"
