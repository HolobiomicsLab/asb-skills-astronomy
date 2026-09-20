---
name: thermochemical-database-querying
description: Use when a task needs temperature-dependent thermochemical properties
  (entropy, enthalpy, Gibbs free energy) of gas or condensate species from the ECCOplanets
  .pkl databases, e.g., as a prerequisite to condensation simulations.
when_to_use_negative:
- The goal is to run or analyse a condensation simulation rather than explore the
  underlying thermochemical data — use the s1simulation.py/s2analyses.py workflow
  instead
- The needed species is absent from the thermochemical database; expand the database
  first via the s4adding_data.py workflow rather than querying
- The task concerns stellar abundance patterns rather than thermochemical data — those
  live in the separate stellar abundance .pkl database
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: data.py
  role: Reads the .pkl thermochemical database files into Python
  repo: https://github.com/AninaTimmermann/ECCOplanets
- name: classes.py (molecule class)
  role: Provides the class whose functions query and explore the thermochemical databases
    (demonstrated in sample s0moleculeDB.py)
  repo: https://github.com/AninaTimmermann/ECCOplanets
- name: matplotlib
  role: Plots the entropy, enthalpy, and Gibbs-energy curves against temperature using
    properties defined in pltStyle.py
- name: pandas / numpy
  role: Support .pkl data loading and Shomate fitting computations
provenance:
  source_task_ids:
  - task_004
  - task_007
  - task_001
  - task_002
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202244850
    title: Revisiting equilibrium condensation and rocky planet compositions
  source_content_hash: sha256:745b436c1851ebfb50bac1604cbd8611ec088ec6f93f0efb75a0c378686a4a75
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AninaTimmermann/ECCOplanets
  tools:
  - ECCOplanets
  - Python
  - classes.py (molecule class)
  - data.py
  - matplotlib
  - numpy
  - pandas
  - pandas / numpy
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# thermochemical-database-querying

## Summary

Query and explore the ECCOplanets thermochemical databases (.pkl files) via the molecule class to retrieve tabulated and fitted Shomate entropy, enthalpy, and Gibbs free energy curves for chemical species (e.g., Al2O3) as a function of temperature.

## When to use

Use when a task needs temperature-dependent thermochemical properties (entropy, enthalpy, Gibbs free energy) of gas or condensate species from the ECCOplanets .pkl databases, e.g., as a prerequisite to condensation simulations.

## When NOT to use

- The goal is to run or analyse a condensation simulation rather than explore the underlying thermochemical data — use the s1simulation.py/s2analyses.py workflow instead
- The needed species is absent from the thermochemical database; expand the database first via the s4adding_data.py workflow rather than querying
- The task concerns stellar abundance patterns rather than thermochemical data — those live in the separate stellar abundance .pkl database

## Inputs

- ECCOplanets thermochemical database (.pkl files, read via data.py)
- Target chemical species identifier (e.g., Al2O3 / corundum)
- Temperature range over which to evaluate or fit the thermochemical curves

## Outputs

- Fitted Shomate coefficients (A–G), piecewise if phase discontinuities are present
- Entropy, enthalpy, and Gibbs free energy (G = H − TS) curves as a function of temperature
- Matplotlib figure (PNG, styled via pltStyle.py) overlaying fitted curves on the tabulated data

## How to apply

Instantiate the molecule class (defined in classes.py) for the target species, which loads the tabulated thermochemical data for that molecule from the .pkl files read in by data.py. Distinguish the two data types shipped in the database: raw tabulated thermochemical data and fitted Shomate data (parameters A–G); when the tabulated curves contain discontinuities — jumps at certain temperatures from reference-state data including multiple phases of a species — fit separate Shomate parameters on either side of the discontinuity to obtain a piecewise continuous thermochemical function. Compute entropy, enthalpy, and Gibbs free energy (G = H − TS) from the fitted Shomate coefficients, then plot the curves against temperature using matplotlib with the figure properties defined in pltStyle.py.

## Related tools

- **data.py** (Reads the .pkl thermochemical database files into Python) — https://github.com/AninaTimmermann/ECCOplanets
- **classes.py (molecule class)** (Provides the class whose functions query and explore the thermochemical databases (demonstrated in sample s0moleculeDB.py)) — https://github.com/AninaTimmermann/ECCOplanets
- **matplotlib** (Plots the entropy, enthalpy, and Gibbs-energy curves against temperature using properties defined in pltStyle.py)
- **pandas / numpy** (Support .pkl data loading and Shomate fitting computations)

## Evaluation signals

- Overlay the fitted curves on the tabulated thermochemical data and confirm the piecewise Shomate fits reproduce the tabulated values across the temperature range
- Verify discontinuities (phase-transition jumps) are handled as separate Shomate segments on either side, yielding a piecewise continuous function
- Check internal consistency of G = H − TS from the fitted Shomate coefficients
- Confirm the output PNG file exists and is a valid figure

## Limitations

- The thermochemical database only covers species included in the .pkl files; expanding it requires the s4adding_data.py workflow
- Tabulated data commonly contain jumps at reference-state phase transitions, so a single global Shomate fit is inadequate and piecewise fitting is required
- ECCOplanets only considers pure minerals at the moment; solid solutions are planned for a later release

## Evidence

- [methods] molecule class instantiation: "In the sample, you create an instance of the molecule class and explore the functionalities of the class."
- [readme] data.py reads .pkl files: "this code only reads the .pkl files containing all relevant data into python"
- [readme] database contents: "data: contains the databases for the thermochemical data and the stellar abundance data"
- [readme] two data types: "You will also look at our different types of data, i.e. tabulated thermochemical data and fitted Shomate data."
