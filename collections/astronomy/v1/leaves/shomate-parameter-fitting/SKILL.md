---
name: shomate-parameter-fitting
description: Use when tabulated enthalpy/entropy data contain discontinuities (multi-phase
  reference states) and you need continuous thermochemical functions S(T), H(T), G(T)
  for Gibbs free energy minimisation.
when_to_use_negative:
- The species' tabulated data are already stored as fitted Shomate data — no refitting
  is needed (distinguish the two data types as in s0moleculeDB.py)
- You want to run or analyse a condensation simulation rather than inspect thermochemical
  data — use s1simulation.py/s2analyses.py instead
- You need solid-solution thermodynamics — ECCOplanets currently considers only pure
  minerals
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: ECCOplanets
  role: Provides data.py (reads .pkl thermochemical databases), the molecule class
    in classes.py, and sample s0moleculeDB.py demonstrating Shomate fitting and plotting
  repo: github.com/AninaTimmermann/ECCOplanets
- name: Python
  role: Language for the fitting; ECCOplanets uses pandas, numpy and matplotlib
- name: matplotlib
  role: Plots the fitted entropy/enthalpy/Gibbs curves against temperature using pltStyle.py
    figure properties
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202244850
    title: Revisiting equilibrium condensation and rocky planet compositions
  source_content_hash: sha256:4b3054c7d8237f388cf99ab963ca3218e4d2a37fbaebc69659f8526374a298af
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ECCOplanets
  - Python
  - matplotlib
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# shomate-parameter-fitting

## Summary

Fits Shomate polynomial coefficients (A–G) to discrete tabulated thermochemical data (e.g., Al2O3 from the ECCOplanets thermochemical .pkl databases), splitting the fit into piecewise segments wherever the tabulated curves show discontinuities such as phase transitions.

## When to use

Use when tabulated enthalpy/entropy data contain discontinuities (multi-phase reference states) and you need continuous thermochemical functions S(T), H(T), G(T) for Gibbs free energy minimisation.

## When NOT to use

- The species' tabulated data are already stored as fitted Shomate data — no refitting is needed (distinguish the two data types as in s0moleculeDB.py)
- You want to run or analyse a condensation simulation rather than inspect thermochemical data — use s1simulation.py/s2analyses.py instead
- You need solid-solution thermodynamics — ECCOplanets currently considers only pure minerals

## Inputs

- Tabulated thermochemical data for a species (e.g., Al2O3) stored in ECCOplanets .pkl files under the data/ folder, read via data.py
- Molecule class instance from classes.py exposing tabulated vs fitted Shomate data

## Outputs

- Piecewise Shomate coefficients A–G (one parameter set per side of each discontinuity)
- Continuous entropy, enthalpy, and Gibbs free energy (G = H − TS) functions of temperature
- Matplotlib figure (PNG) of S, H, G vs temperature styled per pltStyle.py

## How to apply

Load the tabulated thermochemical data for the target species (e.g., Al2O3) from the ECCOplanets thermochemical database, where .pkl files are read via data.py, by instantiating the molecule class. Fit Shomate parameters A–G to the tabulated enthalpies and entropies across the temperature range, splitting the fit into piecewise segments wherever discontinuities (phase transitions) occur in the tabulated curves, so each segment yields a continuous thermochemical function. From the fitted coefficients, compute entropy, enthalpy, and Gibbs free energy using G = H − TS. Plot the S, H, and G curves against temperature with matplotlib, applying the figure properties defined in pltStyle.py. Validate by overlaying the fitted curves on the raw tabulated data and confirming the piecewise fits reproduce the tabulated values across the whole temperature range.

## Related tools

- **ECCOplanets** (Provides data.py (reads .pkl thermochemical databases), the molecule class in classes.py, and sample s0moleculeDB.py demonstrating Shomate fitting and plotting) — github.com/AninaTimmermann/ECCOplanets
- **Python** (Language for the fitting; ECCOplanets uses pandas, numpy and matplotlib)
- **matplotlib** (Plots the fitted entropy/enthalpy/Gibbs curves against temperature using pltStyle.py figure properties)

## Examples

```
python samples/s0moleculeDB.py
```

## Evaluation signals

- Overlay the fitted curves on the tabulated thermochemical data and confirm the piecewise Shomate fits reproduce the tabulated values across the temperature range
- Discontinuities (phase-transition jumps) in the tabulated data are handled by separate Shomate parameter sets on either side, giving a piecewise continuous function
- Gibbs energy curve is internally consistent with G = H − TS over the full temperature range
- Output PNG file exists and is a valid figure rendered with pltStyle.py properties
- Condensation temperatures derived with these thermochemical data should fall within ±5 K of literature values in the code's validation tests

## Limitations

- The code only considers pure minerals at the moment; solid solutions are planned but not yet supported
- Thermochemical databases must contain data for the species; incomplete abundance or data coverage may be scientifically questionable
- Fitted Shomate validity is limited to the temperature range and phase segments of the underlying tabulated data

## Evidence

- [readme] tabulated vs fitted data types: "You will also look at our different types of data, i.e. tabulated thermochemical data and fitted Shomate data."
- [readme] molecule class exploration: "In the sample, you create an instance of the molecule class and explore the functionalities of the class."
- [readme] pkl data loading: "this code only reads the .pkl files containing all relevant data into python"
- [methods] software stack: "The code is built in python and uses the common packages pandas, numpy and matplotlib."
- [other] piecewise fitting of discontinuities: [evidence span withheld — anchor status: not_found]
