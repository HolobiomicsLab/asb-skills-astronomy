---
name: solar-abundance-pattern-configuration
description: Use when setting up equilibrium condensation runs that must start from
  a solar (Solar System) abundance pattern, e.g. to establish a 1e-6 bar baseline
  Tc before varying pressure, metallicity, or C/O.
when_to_use_negative:
- When the goal is a stellar abundance pattern other than solar — use s4adding_data.py
  to expand the stellar abundance database first (note the README's warning about
  incomplete abundance sets lacking Al, Ca, Mg, Si, or O).
- When analysing an already-run simulation (load and post-process it with s2analyses.py
  instead of reconfiguring the simulation).
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: ECCOplanets
  role: Equilibrium condensation code whose simulation.py backbone and s1simulation.py
    sample define the abundance-pattern and simulation parameters
  repo: https://github.com/AninaTimmermann/ECCOplanets
- name: Python
  role: Runtime for ECCOplanets, together with pandas, numpy and matplotlib
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202244850
    title: Revisiting equilibrium condensation and rocky planet compositions
  source_content_hash: sha256:154d832a6e394efcb965c2c68008caa5543d69d05e0c36a89f298eb47aaaf3ca
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# solar-abundance-pattern-configuration

## Summary

Configure an ECCOplanets condensation simulation (via simulation.py following sample s1simulation.py) with the solar elemental abundance pattern, together with temperature range/resolution and molecule selection, so that pressure-variation runs are comparable to the paper's Solar System baseline used to derive 50% condensation temperatures.

## When to use

Use when setting up equilibrium condensation runs that must start from a solar (Solar System) abundance pattern, e.g. to establish a 1e-6 bar baseline Tc before varying pressure, metallicity, or C/O.

## When NOT to use

- When the goal is a stellar abundance pattern other than solar — use s4adding_data.py to expand the stellar abundance database first (note the README's warning about incomplete abundance sets lacking Al, Ca, Mg, Si, or O).
- When analysing an already-run simulation (load and post-process it with s2analyses.py instead of reconfiguring the simulation).

## Inputs

- Solar elemental abundance pattern (Solar System composition from the ECCOplanets stellar abundance database)
- ECCOplanets thermochemical database (.pkl files in the data/ folder)
- Simulation parameters: temperature range and resolution, molecule selection, pressure grid (log-spaced 1e-6 to 1e-1 bar)

## Outputs

- Configured condensation simulation (el_amounts output: molecular amount vs. temperature condensation curves per element/molecule per pressure point)
- Baseline solar-abundance simulation from which per-element 50% condensation temperatures (Tc) are extracted

## How to apply

Start from sample s1simulation.py, which introduces the condensation simulation parameters: temperature range and resolution, molecule selection, and the elemental abundance pattern. Set the abundance pattern to the solar composition available in the ECCOplanets data/ folder (thermochemical and stellar abundance .pkl databases loaded via data.py). For pressure-dependence work, run the Gibbs free energy minimisation simulation at logarithmically spaced pressures from 1e-6 to 1e-1 bar, keeping the abundance pattern fixed so ∆Tc isolates the pressure effect. The simulation backbone in simulation.py contains the mathematical/thermochemical formulation; only edit it if the scientific assumptions themselves must change. Validation anchor: derived Tc values for common planet-building elements should be within ±5 K of literature condensation temperatures.

## Related tools

- **ECCOplanets** (Equilibrium condensation code whose simulation.py backbone and s1simulation.py sample define the abundance-pattern and simulation parameters) — https://github.com/AninaTimmermann/ECCOplanets
- **Python** (Runtime for ECCOplanets, together with pandas, numpy and matplotlib)

## Examples

```
python s1simulation.py  # configure temperature range/resolution, molecule selection, and solar elemental abundance pattern in simulation.py
```

## Evaluation signals

- Derived 50% condensation temperatures for common planet-building elements (Mg, Si, Fe, Al, Ca, Na, Ni) fall within ±5 K of literature values.
- The mean ∆Tc across the 1e-6 to 0.1 bar pressure grid reproduces the reported 357 ± 57 K (≈ +350 K).
- Only the pressure parameter varies between runs; the elemental abundance pattern stays fixed at the solar composition so ∆Tc reflects pressure alone.
- Simulation outputs land in the simulations/ (el_amounts) folder structure and load correctly via the simulation class in s2analyses.py.

## Limitations

- The first release only includes abundance data of F, G, and K stars from the Brewer+2016 database; other stellar types require user-supplied data and consideration of uncertainty.
- Running simulations on very incomplete abundance sets (e.g. missing Al, Ca, Mg, Si, or O) may not be sensible.
- Only pure minerals are considered at the moment; solid solutions are planned but not yet supported.

## Evidence

- [methods] In this sample we introduce the parameters of the condensation simulation, i.e. the temperature range and resolution, the molecule selection, and the elemental abundance pattern.: "In this sample we introduce the parameters of the condensation simulation, i.e. the temperature range and resolution, the molecule selection, and the elemental abundance pattern."
- [abstract] The tested pressure range (10−6 − 0.1 bar) corresponds to ∆Tc ≈ +350 K: "The tested pressure range (10−6 − 0.1 bar) corresponds to ∆Tc ≈ +350 K"
- [readme] this is the backbone of our condensation simulation. It contains the mathematical and thermochemical formulation of our code: "this is the backbone of our condensation simulation. It contains the mathematical and thermochemical formulation of our code"
- [abstract] For common planet-building elements, we derive a Tc that is within ±5 K of literature values: "For common planet-building elements, we derive a Tc that is within ±5 K of literature values"
- [readme] This first release only includes abundance data of F, G, and K stars from the Brewer+2016 database.: "This first release only includes abundance data of F, G, and K stars from the Brewer+2016 database."
