---
name: stellar-abundance-simulation-input
description: Use when a condensation simulation needs a realistic, non-solar elemental
  abundance pattern taken from an observed stellar catalogue (e.g. quality-selected
  Brewer+2016 stars with S/N > 100 and log g > 3.5) instead of Solar System parameters.
when_to_use_negative:
- The target star lacks reported abundances for key planet-building elements such
  as Al, Ca, Mg, Si, or O — the README explicitly questions the sensibility of running
  simulations on very incomplete abundance data sets
- The goal is gas-only chemistry without condensates, e.g. when a code limited to
  gas-phase simulations such as TEA would suffice
- Solar System reference parameters are sufficient, so no host-star-specific abundance
  pattern is needed
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: ECCOplanets
  role: Python code whose data.py reads the .pkl thermochemical and stellar abundance
    databases and whose simulation.py runs the Gibbs-free-energy-minimisation condensation
    simulation with the star-specific abundance pattern
  repo: https://github.com/AninaTimmermann/ECCOplanets
- name: Brewer+2016 stellar abundance database
  role: Source of F, G, and K stellar abundance patterns packaged with ECCOplanets;
    the quality-selected subsample (S/N > 100, log g > 3.5) supplies realistic input
    patterns
- name: abundance_converter.py
  role: Functions to convert between different stellar elemental abundance formats
    when adding new stars via s4adding_data.py
  repo: https://github.com/AninaTimmermann/ECCOplanets
- name: pandas
  role: Common package used by the code to handle the tabulated stellar abundance
    and thermochemical data
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1051/0004-6361/202244850
    title: Revisiting equilibrium condensation and rocky planet compositions
  source_content_hash: sha256:26d5b2b5212c333bd12dd67d0a108554db9864107ed85119e22208b72b488012
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AninaTimmermann/ECCOplanets
  tools:
  - Brewer+2016 stellar abundance database
  - ECCOplanets
  - Python
  - abundance_converter.py
  - matplotlib
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# stellar-abundance-simulation-input

## Summary

Prepare a star-specific elemental abundance pattern for an ECCOplanets condensation simulation by loading a packaged stellar abundance database (e.g. Brewer+2016 F/G/K stars) with data.py and passing it into simulation.py alongside the temperature range, resolution, and molecule selection.

## When to use

Use when a condensation simulation needs a realistic, non-solar elemental abundance pattern taken from an observed stellar catalogue (e.g. quality-selected Brewer+2016 stars with S/N > 100 and log g > 3.5) instead of Solar System parameters.

## When NOT to use

- The target star lacks reported abundances for key planet-building elements such as Al, Ca, Mg, Si, or O — the README explicitly questions the sensibility of running simulations on very incomplete abundance data sets
- The goal is gas-only chemistry without condensates, e.g. when a code limited to gas-phase simulations such as TEA would suffice
- Solar System reference parameters are sufficient, so no host-star-specific abundance pattern is needed

## Inputs

- Packaged .pkl stellar abundance database (Brewer+2016 F, G, K stars) and .pkl thermochemical database, read via data.py
- Star identifier / abundance pattern for the quality-selected subsample (S/N > 100, log g > 3.5)
- Simulation parameters: temperature range and resolution, molecule selection, elemental abundance pattern (s1simulation.py)

## Outputs

- Condensation simulation result (molecule-vs-temperature curves) computed with the star's abundance pattern, consumable by the classes.py simulation class
- Expanded stellar abundance database entries when new stars are added via s4adding_data.py

## How to apply

Load the packaged .pkl thermochemical and stellar abundance databases via data.py; the first release covers F, G, and K stars from the Brewer+2016 database, so restrict to the quality-selected subsample (S/N > 100, log g > 3.5). In s1simulation.py, set the simulation parameters — temperature range and resolution, molecule selection, and the star-specific elemental abundance pattern — then run the simulation via simulation.py, which contains the mathematical and thermochemical formulation and the optimisation parameters. Before running, check that the star's abundance data are complete for the planet-building elements, since the README warns against simulations on very incomplete abundance sets (e.g. missing Al, Ca, Mg, Si, or O). If the abundance format of a new star differs, use abundance_converter.py (via s4adding_data.py) to convert it and expand the stellar abundance database.

## Related tools

- **ECCOplanets** (Python code whose data.py reads the .pkl thermochemical and stellar abundance databases and whose simulation.py runs the Gibbs-free-energy-minimisation condensation simulation with the star-specific abundance pattern) — https://github.com/AninaTimmermann/ECCOplanets
- **Brewer+2016 stellar abundance database** (Source of F, G, and K stellar abundance patterns packaged with ECCOplanets; the quality-selected subsample (S/N > 100, log g > 3.5) supplies realistic input patterns)
- **abundance_converter.py** (Functions to convert between different stellar elemental abundance formats when adding new stars via s4adding_data.py) — https://github.com/AninaTimmermann/ECCOplanets
- **pandas** (Common package used by the code to handle the tabulated stellar abundance and thermochemical data)

## Examples

```
import data; db = data.load(); # in s1simulation.py: set temperature range/resolution, molecule selection, and the star-specific elemental abundance pattern, then run simulation.py
```

## Evaluation signals

- The loaded pattern corresponds to the quality-selected subsample (S/N > 100, log g > 3.5) of the Brewer+2016 database
- The star's abundance data include the key planet-building elements (Al, Ca, Mg, Si, O); otherwise the simulation should be reconsidered
- Derived condensation temperatures for common planet-building elements agree with literature values within ±5 K
- The simulation reproduces expected trend behaviour, e.g. rising Tc with metallicity and ΔTc ≈ −100 K for a C/O increase from 0.1 to 0.7, when the abundance pattern is varied
- The run uses simulation.py's optimisation parameters unchanged unless the scientific assumptions of the condensation simulation are deliberately changed

## Limitations

- The first release only includes abundance data of F, G, and K stars from the Brewer+2016 database; other stellar types require careful consideration of uncertainty
- Running simulations on very incomplete abundance data sets (e.g. missing Al, Ca, Mg, Si, or O) may not be sensible
- Only pure minerals are considered at the moment; solid solutions are planned for a later point

## Evidence

- [methods] Load the Brewer+2016 stellar abundance patterns for the quality-selected subsample (S/N > 100, log g > 3.5, ~100 F, G and K stars) using data.py: [evidence span withheld — anchor status: not_found]
- [methods] In this sample we introduce the parameters of the condensation simulation: "In this sample we introduce the parameters of the condensation simulation, i.e. the temperature range and resolution, the molecule selection, and the elemental abundance pattern."
- [readme] This first release only includes abundance data of F, G, and K stars from the Brewer+2016 database.: "This first release only includes abundance data of F, G, and K stars from the Brewer+2016 database. There is absolutely no reason, though, not to include data of other stellar types, just be sure to"
- [readme] you might need to think about the sensibility of running simulations on very incomplete abundance data sets: "Also you might need to think about the sensibility of running simulations on very incomplete abundance data sets (e.g. if Al, Ca, Mg, Si, or O data is not reported)."
- [readme] includes functions to convert between different stellar elemental abundance formats: "includes functions to convert between different stellar elemental abundance formats (used in s4adding_data.py)"
