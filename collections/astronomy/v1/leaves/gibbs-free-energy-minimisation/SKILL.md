---
name: gibbs-free-energy-minimisation
description: Use when condensation sequences, 50% condensation temperatures of elements/species,
  or equilibrium abundances as a function of temperature are needed for a given elemental
  abundance pattern (e.g. solar, [M/H]-scaled, or star-specific Brewer+2016) at a
  fixed total pressure.
when_to_use_negative:
- Gas-phase-only equilibrium is sufficient — the article notes the TEA code 'is limited
  to gas-phase simulations', but this skill's value is tracking condensates
- Solid solutions are required — the code 'only consider[s] pure minerals at the moment'
  and solid solutions are planned future work
- The star's abundance set is very incomplete (e.g. Al, Ca, Mg, Si, or O not reported)
  — the README warns to consider the sensibility of running simulations on such data
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3314
tools:
- name: ECCOplanets
  role: Provides simulation.py, the backbone containing the mathematical/thermochemical
    formulation and optimiser parameters of the Gibbs minimisation condensation simulation
  repo: https://github.com/AninaTimmermann/ECCOplanets
- name: Python
  role: Language in which the code and its optimisation routine are implemented
  repo: https://github.com/AninaTimmermann/ECCOplanets
- name: numpy
  role: Numeric arrays for the Gibbs minimisation problem (gradient/Hessian) and temperature
    grid
- name: pandas
  role: Reads the .pkl thermochemical and stellar abundance databases and stores simulation
    results
- name: matplotlib
  role: Plots molecule/element abundance curves vs temperature using figure properties
    from pltStyle.py
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_006
  - task_001
  - task_008
  - task_007
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202244850
    title: Revisiting equilibrium condensation and rocky planet compositions
  source_content_hash: sha256:3bf814959b9589d1249ac5dd21dba52e4d2f6397ba085b72e479fa3cc01b5a57
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# gibbs-free-energy-minimisation

## Summary

Compute temperature-dependent equilibrium composition of a protoplanetary disk by minimising Gibbs free energy over gas-phase and pure-condensate species, as implemented in the ECCOplanets simulation.py with a trust-region solver under non-negativity and number-balance constraints.

## When to use

Use when condensation sequences, 50% condensation temperatures of elements/species, or equilibrium abundances as a function of temperature are needed for a given elemental abundance pattern (e.g. solar, [M/H]-scaled, or star-specific Brewer+2016) at a fixed total pressure.

## When NOT to use

- Gas-phase-only equilibrium is sufficient — the article notes the TEA code 'is limited to gas-phase simulations', but this skill's value is tracking condensates
- Solid solutions are required — the code 'only consider[s] pure minerals at the moment' and solid solutions are planned future work
- The star's abundance set is very incomplete (e.g. Al, Ca, Mg, Si, or O not reported) — the README warns to consider the sensibility of running simulations on such data

## Inputs

- ECCOplanets thermochemical database (.pkl files read via data.py)
- Stellar elemental abundance pattern (.pkl; e.g. Lodders2003a solar pattern, synthetic [M/H]-scaled or C/O-varied patterns, or Brewer+2016 F/G/K star abundances)
- Simulation parameters: temperature range and resolution, molecule selection, total pressure (e.g. 1e-4 bar)

## Outputs

- condensation_simulation.pkl with relative molar amounts of each gas and solid species at each temperature step
- 50% condensation temperatures per element/species (e.g. condensation_temperatures.csv, condensation_temperatures_co_variation.csv)
- Molecule/element abundance-vs-temperature plots (matplotlib, pltStyle.py)

## How to apply

Load the thermochemical .pkl databases and the chosen stellar elemental abundance pattern via data.py/pandas, then configure the simulation in the style of sample s1simulation.py: temperature range (e.g. 2000 K down to 300 K), resolution, molecule selection (default: most common chemical species), and total pressure (e.g. 1e-4 bar). At each temperature step, formulate the constrained convex Gibbs minimisation over the selected gas and pure-condensate species (Eqs. 15-18) supplying the analytic gradient and Hessian, enforce non-negativity and number-balance constraints, and solve with a trust-region optimisation; seed each step with the solution from the previous temperature, which is the key stabilising trick for tracking the condensation sequence. Store the relative molar amounts of every gas and solid species per temperature as a simulation result, then post-process with s2analyses.py/classses.py: crop the gas phase, recompute a section to remove numerical glitches, and smooth the molecule-abundance curves before extracting 50% condensation temperatures. Validate against benchmarks: solar-abundance condensation temperatures must match Lodders2003a/Condor values within ±5 K (e.g. Al 1643 K, Ti 1575 K, Ca 1512 K, Mg 1331 K, Fe 1329 K, Si 1305 K at 1e-4 bar).

## Related tools

- **ECCOplanets** (Provides simulation.py, the backbone containing the mathematical/thermochemical formulation and optimiser parameters of the Gibbs minimisation condensation simulation) — https://github.com/AninaTimmermann/ECCOplanets
- **Python** (Language in which the code and its optimisation routine are implemented) — https://github.com/AninaTimmermann/ECCOplanets
- **numpy** (Numeric arrays for the Gibbs minimisation problem (gradient/Hessian) and temperature grid)
- **pandas** (Reads the .pkl thermochemical and stellar abundance databases and stores simulation results)
- **matplotlib** (Plots molecule/element abundance curves vs temperature using figure properties from pltStyle.py)

## Examples

```
run s1simulation.py with temperature range 2000–300 K, molecule selection = most common species, solar abundance pattern, p = 1e-4 bar; then post-process via s2analyses.py
```

## Evaluation signals

- Derived solar-abundance condensation temperatures for common planet-building elements fall within ±5 K of Lodders2003a/Condor literature values (Al 1643 K, Ti 1575 K, Ca 1512 K, Mg 1331 K, Fe 1329 K, Si 1305 K at p = 1e-4 bar)
- Species-vs-species comparison against Table 1 benchmark agrees mostly within ±50 K, with average offset slightly lower due to the 50% condensation-temperature definition
- Each temperature step's solution converges under trust-region optimisation with non-negativity and number-balance constraints satisfied (feasible molar amounts)
- Output pkl contains molar amounts for all selected gas and solid species at every temperature step of the configured grid
- Post-processing (crop gas phase, recompute section, smooth) removes numerical glitches such that extracted 50% condensation points are stable

## Limitations

- Validation criterion of ±5 K applies only to common planet-building elements; Table-1-style species comparisons agree only within ~±50 K due to differing 50% condensation definitions
- Condensation temperatures are sensitive to total pressure (10^-6–0.1 bar spans ∆Tc ≈ +350 K), metallicity (∆Tc ≈ +100 K for -0.3 to +0.4 dex) and C/O (∆Tc ≈ -100 K from 0.1 to 0.7), so parameters must be stated
- Only pure minerals are considered; solid solutions are not yet supported
- Raw simulation output contains numerical glitches requiring crop/recompute/smooth post-processing before condensation temperatures are extracted

## Evidence

- [methods] trust-region with previous-step warm start: [evidence span withheld — anchor status: not_found]
- [abstract] validation criterion: "For common planet-building elements, we derive a Tc that is within ±5 K of literature values"
- [readme] simulation parameters scope: "In this sample we introduce the parameters of the condensation simulation, i.e. the temperature range and resolution, the molecule selection, and the elemental abundance pattern."
- [readme] simulation.py role: "this is the backbone of our condensation simulation. It contains the mathematical and thermochemical formulation of our code and the parameters of the optimisation procedure"
- [methods] post-processing steps: "improving the simulation result by recomputing a section, removing numerical glitches, and smoothing the curve"
