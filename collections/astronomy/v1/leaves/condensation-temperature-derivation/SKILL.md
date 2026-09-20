---
name: condensation-temperature-derivation
description: Use when a T-p-dependent condensation simulation (Gibbs minimisation
  over gas + pure-condensate species) has been run for a stellar abundance pattern
  at a fixed pressure (e.g. 1e-4 bar) and you need to extract and validate per-element
  condensation temperatures against literature benchmarks.
when_to_use_negative:
- The input chemistry is gas-phase only (e.g. hot-atmosphere equilibrium with TEA-style
  codes) — no condensate 50% point exists to extract
- Simulations require solid-solution minerals rather than pure minerals — the ECCOplanets
  code considers only pure minerals at present
- The stellar abundance pattern is highly incomplete for key planet-building elements
  (Al, Ca, Mg, Si, O data missing) — running such simulations is flagged as scientifically
  unsound in the README
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: ECCOplanets
  role: 'Core code: simulation.py implements the Gibbs minimisation; s1simulation.py
    configures and runs the condensation simulation; s2analyses.py post-processes
    and extracts condensation temperatures'
  repo: github.com/AninaTimmermann/ECCOplanets
- name: Python
  role: Language in which the code and analysis samples are built and run
- name: pandas
  role: Reads the .pkl thermochemical and abundance databases and writes the condensation_temperatures.csv
    benchmark table
- name: numpy
  role: Numerical backbone of the constrained Gibbs minimisation (gradient, Hessian,
    trust-region solve)
- name: matplotlib
  role: Plots the molecule-abundance vs temperature curves used for smoothing and
    inspection
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202244850
    title: Revisiting equilibrium condensation and rocky planet compositions
  source_content_hash: sha256:ed7fe365de6b5bb22b456567572d38ab2ac356b3aeb9409f56297bed855028c3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ECCOplanets
  - Python
  - matplotlib
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# condensation-temperature-derivation

## Summary

Derive 50%-condensation temperatures of common rocky species (corundum, perovskite, gehlenite, spinel, anorthite, iron, forsterite, diopside, enstatite) from an ECCOplanets Gibbs free energy minimisation condensation simulation, and benchmark them against the Lodders2003a Condor-code values. It turns raw temperature-resolved molar abundance curves into per-element condensation temperatures for a given pressure and stellar abundance pattern.

## When to use

Use when a T-p-dependent condensation simulation (Gibbs minimisation over gas + pure-condensate species) has been run for a stellar abundance pattern at a fixed pressure (e.g. 1e-4 bar) and you need to extract and validate per-element condensation temperatures against literature benchmarks.

## When NOT to use

- The input chemistry is gas-phase only (e.g. hot-atmosphere equilibrium with TEA-style codes) — no condensate 50% point exists to extract
- Simulations require solid-solution minerals rather than pure minerals — the ECCOplanets code considers only pure minerals at present
- The stellar abundance pattern is highly incomplete for key planet-building elements (Al, Ca, Mg, Si, O data missing) — running such simulations is flagged as scientifically unsound in the README

## Inputs

- Thermochemical database (.pkl files) containing tabulated and Shomate-fitted data per molecule
- Stellar elemental abundance pattern (e.g. Solar System; database covers F, G, K stars from Brewer+2016)
- Simulation configuration: temperature range and resolution, molecule selection, total pressure (e.g. 1e-4 bar)
- Raw simulation result (condensation_simulation.pkl): relative molar amounts of each gas and solid species per temperature step

## Outputs

- Condensation temperatures (50% condensation point) per element/species for the target abundance pattern and pressure
- Benchmark comparison table (condensation_temperatures.csv) of derived vs Lodders2003a Condor values
- Cleaned and smoothed molecule-abundance vs temperature curves from the analysed simulation object

## How to apply

Load the thermochemical (.pkl) and stellar abundance databases via data.py and configure the simulation class in s1simulation.py with the temperature range/resolution, a molecule selection restricted to the most common chemical species, and the solar elemental abundance pattern. Run the Gibbs free energy minimisation (simulation.py) at the target total pressure — 1e-4 bar for the Solar System benchmark — using the trust-region optimiser (non-negativity and number-balance constraints, analytic gradient and Hessian, previous temperature step's solution as initial guess). Before extracting temperatures, improve the raw result with s2analyses.py: crop the gas phase, recompute a section to remove numerical glitches, and smooth the molecule-abundance curves. Extract the condensation temperature of each target element/species as its 50% condensation point from the analysed simulation object. Validate by comparing against the Lodders2003a Condor benchmark: agreement must fall within ±5 K per the paper's criterion, with deviations explained by the differing 50%-condensation-temperature definition (results average slightly lower but mostly within ±50 K).

## Related tools

- **ECCOplanets** (Core code: simulation.py implements the Gibbs minimisation; s1simulation.py configures and runs the condensation simulation; s2analyses.py post-processes and extracts condensation temperatures) — github.com/AninaTimmermann/ECCOplanets
- **Python** (Language in which the code and analysis samples are built and run)
- **pandas** (Reads the .pkl thermochemical and abundance databases and writes the condensation_temperatures.csv benchmark table)
- **numpy** (Numerical backbone of the constrained Gibbs minimisation (gradient, Hessian, trust-region solve))
- **matplotlib** (Plots the molecule-abundance vs temperature curves used for smoothing and inspection)

## Examples

```
run samples/s1simulation.py to run the condensation simulation at p=1e-4 bar with the solar abundance pattern, then run samples/s2analyses.py to crop, recompute, smooth, and extract per-element condensation temperatures
```

## Evaluation signals

- Derived condensation temperatures for common planet-building elements fall within ±5 K of the published Lodders2003a Condor values (the paper's stated benchmark criterion)
- Per-species deviations are mostly within ±50 K, with systematically slightly lower values attributable to the different 50% condensation-temperature definition
- Molecule-abundance curves are free of numerical glitches and are smooth after cropping/recompute/smoothing in s2analyses.py
- condensation_temperatures.csv contains rows for all nine target species (corundum, perovskite, gehlenite, spinel, anorthite, iron, forsterite, diopside, enstatite) with paired benchmark values
- Temperature-step continuity holds: each step's solution (used as the next initial guess) does not produce discontinuous jumps in molar amounts

## Limitations

- Condensation temperatures are sensitive to total pressure and metallicity: 10^-6–0.1 bar corresponds to ΔTc ≈ +350 K and −0.3 ≤ [M/H] ≤ 0.4 to ΔTc ≈ +100 K, so benchmarking is only valid at matching parameters
- Increasing C/O from 0.1 to 0.7 decreases Tc by ΔTc ≈ −100 K; abundance-pattern choice matters and must be matched to the benchmark
- Only pure minerals are considered; solid solutions are planned but not implemented in this release
- The 50%-condensation-temperature definition differs from Condor's, producing a small systematic offset (average slightly lower values)

## Evidence

- [abstract] Validation criterion for benchmark comparison: "For common planet-building elements, we derive a Tc that is within ±5 K of literature values"
- [methods] Simulation setup parameters and sample workflow: "In this sample we introduce the parameters of the condensation simulation, i.e. the temperature range and resolution, the molecule selection, and the elemental abundance pattern."
- [methods] Result-improving filter steps before temperature extraction: "improving the simulation result by recomputing a section, removing numerical glitches, and smoothing the curve"
- [abstract] Gibbs minimisation formulation of the condensation sequence: "Our T-p-dependent chemical equilibrium model is based on a Gibbs free energy minimisation."
- [readme] Data-loading layer and database formats from the README: "data.py — this code only reads the .pkl files containing all relevant data into python"
- [results] Solver stability rationale for the optimisation procedure: [evidence span withheld — anchor status: not_found]
