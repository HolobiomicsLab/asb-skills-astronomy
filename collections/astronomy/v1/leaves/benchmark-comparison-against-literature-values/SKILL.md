---
name: benchmark-comparison-against-literature-values
description: Use when a newly implemented equilibrium or simulation model must be
  validated before production runs, and a published reference dataset (e.g. literature
  condensation temperatures) exists for identical input parameters.
when_to_use_negative:
- No published benchmark under matched input parameters (pressure, abundance pattern)
  exists — differences would conflate code error with parameter mismatch
- The goal is exploring parameter sensitivity (∆Tc vs pressure, metallicity, C/O)
  rather than validating the code itself
- Definitional conventions (e.g. 50% condensation point) differ between the new code
  and the reference, and no offset correction/interpretation is planned
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: ECCOplanets
  role: Condensation simulation code whose derived 50% condensation temperatures are
    the values being benchmarked
  repo: https://github.com/AninaTimmermann/ECCOplanets
- name: Condor
  role: External benchmark code providing the published Lodders2003a condensation-temperature
    reference values
- name: pandas
  role: Tabulates derived vs benchmark condensation temperatures and writes condensation_temperatures.csv
- name: s2analyses.py
  role: Crops the gas phase, recomputes sections, removes numerical glitches, and
    smooths curves so extracted condensation temperatures are not artefacts
  repo: https://github.com/AninaTimmermann/ECCOplanets
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202244850
    title: Revisiting equilibrium condensation and rocky planet compositions
  source_content_hash: sha256:bca43c5ba3ed1a7c3517cbd016c01206a0d8d73239a1fe27f8406c75b3856072
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
  - s2analyses.py
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# benchmark-comparison-against-literature-values

## Summary

Validates a new physics/chemistry simulation code by comparing its derived outputs (here: 50% condensation temperatures from ECCOplanets Gibbs free energy minimisation) against established benchmark values (Lodders2003a Condor code), quantifying agreement per species and attributing systematic offsets.

## When to use

Use when a newly implemented equilibrium or simulation model must be validated before production runs, and a published reference dataset (e.g. literature condensation temperatures) exists for identical input parameters.

## When NOT to use

- No published benchmark under matched input parameters (pressure, abundance pattern) exists — differences would conflate code error with parameter mismatch
- The goal is exploring parameter sensitivity (∆Tc vs pressure, metallicity, C/O) rather than validating the code itself
- Definitional conventions (e.g. 50% condensation point) differ between the new code and the reference, and no offset correction/interpretation is planned

## Inputs

- ECCOplanets simulation object for Solar System parameters (solar abundance pattern, p = 1e-4 bar) from s1simulation.py, analysed via s2analyses.py
- Published benchmark table of condensation temperatures (Lodders2003a Condor-code values) for the target rocky species

## Outputs

- condensation_temperatures.csv with derived vs Lodders2003a benchmark values per species
- Per-species agreement assessment (e.g. within ±5 K of the published Condor values)

## How to apply

Run the new code under the same input parameters as the published benchmark — here ECCOplanets with Solar System abundances at p = 1e-4 bar, after gas-phase cropping, section recomputation, glitch removal, and curve smoothing via s2analyses.py. Extract the target quantities per species (corundum, perovskite, gehlenite, spinel, anorthite, iron, forsterite, diopside, enstatite) using the 50% condensation point definition. Tabulate derived vs benchmark values (e.g. condensation_temperatures.csv) and compute per-species differences. Accept the implementation when common planet-building elements fall within ±5 K of the literature values; where systematic offsets appear (ECCOplanets averages slightly lower, mostly within ±50 K), attribute them to definitional differences (the 50% condensation-temperature definition) rather than code error.

## Related tools

- **ECCOplanets** (Condensation simulation code whose derived 50% condensation temperatures are the values being benchmarked) — https://github.com/AninaTimmermann/ECCOplanets
- **Condor** (External benchmark code providing the published Lodders2003a condensation-temperature reference values)
- **pandas** (Tabulates derived vs benchmark condensation temperatures and writes condensation_temperatures.csv)
- **s2analyses.py** (Crops the gas phase, recomputes sections, removes numerical glitches, and smooths curves so extracted condensation temperatures are not artefacts) — https://github.com/AninaTimmermann/ECCOplanets

## Examples

```
# run s1simulation.py (Solar System parameters, p=1e-4 bar), then s2analyses.py; compare vs Lodders2003a and write:
simulation.Tc_compare(benchmark='Lodders2003a', output='condensation_temperatures.csv')
```

## Evaluation signals

- Derived condensation temperatures for common planet-building elements are within ±5 K of the published Lodders2003a Condor values
- Agreement holds per species for corundum, perovskite, gehlenite, spinel, anorthite, iron, forsterite, diopside, and enstatite
- Any systematic offset is explainable: ECCOplanets values slightly lower on average, mostly within ±50 K, due to the different 50% condensation-temperature definition
- condensation_temperatures.csv contains both derived and benchmark columns for every target species
- Simulation post-processing (cropping, recomputation, glitch removal, smoothing) was applied before extraction so no numerical glitches contaminate the extracted temperatures

## Limitations

- Small systematic offsets (up to ~50 K, slightly lower on average) arise from the different 50% condensation-temperature definition, not code error
- ECCOplanets only considers pure minerals at the moment; solid solutions are planned but not yet supported, which may limit comparability for species affected by solid-solution behaviour
- The code is intended as a simplified starting point giving approximate planetary compositions, not a replacement for full-featured codes like Condor

## Evidence

- [abstract] Validation criterion: "For common planet-building elements, we derive a Tc that is within ±5 K of literature values"
- [methods] Post-processing before extraction: "improving the simulation result by recomputing a section, removing numerical glitches, and smoothing the curve"
- [results] Systematic offset attribution: [evidence span withheld — anchor status: not_found]
- [readme] Pure-minerals-only caveat: "We only consider pure minerals at the moment, but there are plans to add solid solutions at a later point."
