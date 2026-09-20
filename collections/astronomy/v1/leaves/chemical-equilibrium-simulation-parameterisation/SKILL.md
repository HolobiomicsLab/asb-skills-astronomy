---
name: chemical-equilibrium-simulation-parameterisation
description: Use when condensation temperatures (Tc) or molecule/element abundance-vs-temperature
  curves must be derived from a Gibbs free energy minimisation under a defined temperature
  grid, total pressure (e.g. 1e-4 bar), and stellar elemental abundance pattern.
when_to_use_negative:
- The problem is gas-phase chemistry only — a code limited to gas-phase simulations
  (like TEA) may suffice and the full condensation machinery is unnecessary
- Solid solutions are required — ECCOplanets only considers pure minerals at the moment
- The stellar abundance data is very incomplete (e.g. Al, Ca, Mg, Si, or O data not
  reported) — the README warns to consider the sensibility of running simulations
  on such datasets
edam_operation: http://edamontology.org/operation_3679
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: ECCOplanets
  role: Gibbs free energy minimisation condensation engine; simulation.py holds the
    mathematical/thermochemical formulation, data.py loads .pkl databases, s1simulation.py
    and s2analyses.py drive setup, refinement, and Tc extraction
  repo: https://github.com/AninaTimmermann/ECCOplanets
- name: Python
  role: Runtime with pandas, numpy, and matplotlib for running the equilibrium simulation,
    post-processing curves, and producing the condensation_temperatures.csv table
- name: Condor
  role: External benchmark code (Lodders2003a) whose published condensation temperatures
    validate the derived Tc values
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202244850
    title: Revisiting equilibrium condensation and rocky planet compositions
  source_content_hash: sha256:1004cb5f79f7b6fc6312b2d4ddd063ef214d403ad22d044101a53011478f05f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AninaTimmermann/ECCOplanets
  tools:
  - Condor
  - ECCOplanets
  - Python
  - matplotlib
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_3679
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# chemical-equilibrium-simulation-parameterisation

## Summary

Parameterises and runs a T-p-dependent chemical equilibrium condensation simulation via Gibbs free energy minimisation (ECCOplanets) for a stellar abundance pattern and disk pressure, then refines the raw output to extract 50% condensation temperatures of planet-building elements and molecules.

## When to use

Use when condensation temperatures (Tc) or molecule/element abundance-vs-temperature curves must be derived from a Gibbs free energy minimisation under a defined temperature grid, total pressure (e.g. 1e-4 bar), and stellar elemental abundance pattern.

## When NOT to use

- The problem is gas-phase chemistry only — a code limited to gas-phase simulations (like TEA) may suffice and the full condensation machinery is unnecessary
- Solid solutions are required — ECCOplanets only considers pure minerals at the moment
- The stellar abundance data is very incomplete (e.g. Al, Ca, Mg, Si, or O data not reported) — the README warns to consider the sensibility of running simulations on such datasets

## Inputs

- Thermochemical database (.pkl) of tabulated and fitted Shomate data, read via data.py
- Stellar abundance database (Brewer+2016 F, G, and K stars in the first release)
- Simulation parameters: temperature range and resolution, molecule selection, elemental abundance pattern (set in s1simulation.py)
- Total pressure of the condensation run (e.g. 1e-4 bar for Solar System parameters)

## Outputs

- Molecule/element abundance-vs-temperature curves (line plots) from the simulation object
- 50% condensation temperatures (Tc) of elements and molecules
- condensation_temperatures.csv comparing derived Tc against Lodders2003a Condor benchmark values
- Analysed simulation object with cropped, recomputed, and smoothed curves

## How to apply

Load the thermochemical (.pkl) and stellar abundance databases via data.py, then set up the simulation class following s1simulation.py, specifying the temperature range and resolution, the molecule selection (restricted to the most common chemical species), and the elemental abundance pattern. Run the Gibbs free energy minimisation implemented in simulation.py — the mathematical and thermochemical backbone where scientific assumptions of the condensation simulation are changed — at the chosen total pressure (1e-4 bar for Solar System parameters). Refine the raw result with s2analyses.py utilities: crop the gas phase, recompute a section to remove numerical glitches, and smooth the molecule-abundance curves. Extract the 50% condensation temperature of each target species (e.g. corundum, perovskite, gehlenite, spinel, anorthite, iron, forsterite, diopside, enstatite) and compare against the Lodders2003a Condor benchmark before writing condensation_temperatures.csv.

## Related tools

- **ECCOplanets** (Gibbs free energy minimisation condensation engine; simulation.py holds the mathematical/thermochemical formulation, data.py loads .pkl databases, s1simulation.py and s2analyses.py drive setup, refinement, and Tc extraction) — https://github.com/AninaTimmermann/ECCOplanets
- **Python** (Runtime with pandas, numpy, and matplotlib for running the equilibrium simulation, post-processing curves, and producing the condensation_temperatures.csv table)
- **Condor** (External benchmark code (Lodders2003a) whose published condensation temperatures validate the derived Tc values)

## Examples

```
python samples/s1simulation.py && python samples/s2analyses.py
```

## Evaluation signals

- Derived condensation temperatures of common planet-building elements fall within ±5 K of literature (Condor, Lodders2003a) values — the paper's benchmark criterion
- Per-species Tc agreement is mostly within ±50 K, with values on average slightly lower due to the different 50% condensation-temperature definition — larger deviations indicate a setup error
- After refinement, molecule-abundance curves are free of numerical glitches (cropped gas phase, recomputed section, smoothed curves)
- condensation_temperatures.csv contains Tc for the nine target species (corundum, perovskite, gehlenite, spinel, anorthite, iron, forsterite, diopside, enstatite)

## Limitations

- Sensitivity results were validated only over a pressure range of 1e-6 to 0.1 bar (ΔTc ≈ +350 K) and metallicity −0.3 ≤ [M/H] ≤ 0.4 (ΔTc ≈ +100 K); outside these, behaviour is not benchmarked
- Only pure minerals are considered; solid solutions are planned but not yet implemented
- The first release only includes abundance data of F, G, and K stars from the Brewer+2016 database; other stellar types require adding data and considering uncertainty
- The 50% condensation-temperature definition differs from Lodders2003a, producing systematically slightly lower Tc on average

## Evidence

- [abstract] Our T-p-dependent chemical equilibrium model is based on a Gibbs free energy minimisation.: "Our T-p-dependent chemical equilibrium model is based on a Gibbs free energy minimisation."
- [abstract] For common planet-building elements, we derive a Tc that is within ±5 K of literature values: "For common planet-building elements, we derive a Tc that is within ±5 K of literature values"
- [readme] In this sample we introduce the parameters of the condensation simulation, i.e. the temperature range and resolution, the molecule selection, and the elemental abundance pattern.: "In this sample we introduce the parameters of the condensation simulation, i.e. the temperature range and resolution, the molecule selection, and the elemental abundance pattern."
- [readme] this is the backbone of our condensation simulation. It contains the mathematical and thermochemical formulation of our code: "this is the backbone of our condensation simulation. It contains the mathematical and thermochemical formulation of our code"
- [methods] improving the simulation result by recomputing a section, removing numerical glitches, and smoothing the curve: "improving the simulation result by recomputing a section, removing numerical glitches, and smoothing the curve"
