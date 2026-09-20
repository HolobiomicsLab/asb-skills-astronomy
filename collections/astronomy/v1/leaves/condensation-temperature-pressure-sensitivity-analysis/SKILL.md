---
name: condensation-temperature-pressure-sensitivity-analysis
description: Use when you need to determine how equilibrium condensation temperatures
  of rock-forming elements depend on disk pressure (e.g. testing whether element volatility
  defined at Solar System pressure applies to other disks), or to reproduce a ∆Tc
  vs. pressure sensitivity table.
when_to_use_negative:
- The question concerns abundance-pattern sensitivity (e.g. C/O variation from 0.1
  to 0.7 giving ∆Tc ≈ −100 K) or metallicity ([M/H] −0.3 to 0.4 giving ∆Tc ≈ +100
  K) rather than disk pressure — vary those parameters instead.
- You need full planet bulk compositions, devolatilisation, or feeding-zone (FZ) model
  outputs — use the s3planet_composition.py workflow, not a Tc-vs-pressure sweep.
- Your code cannot handle condensed phases (e.g. gas-phase-only codes like TEA) —
  condensation temperatures of rock-forming elements require Gibbs minimisation over
  solid species.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3318
tools:
- name: ECCOplanets
  role: Open-source Python code performing the Gibbs free energy minimisation condensation
    simulation across the pressure grid; provides simulation.py, s1simulation.py,
    and s2analyses.py workflow
  repo: https://github.com/AninaTimmermann/ECCOplanets
- name: Python
  role: Runtime environment for running simulations and analysis scripts
- name: pandas
  role: Tabulating per-element Tc and ∆Tc results across pressure runs
- name: numpy
  role: Numerical handling of molecular amount vs. temperature arrays and Tc extraction
- name: matplotlib
  role: Plotting molecular amount vs. temperature condensation curves used to inspect
    pressure dependence
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202244850
    title: Revisiting equilibrium condensation and rocky planet compositions
  source_content_hash: sha256:5cc63ee12f224e0b6ec7e35247ad9780e2d3393d2dbca64e12bd51096120acde
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
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# condensation-temperature-pressure-sensitivity-analysis

## Summary

Quantifies how the 50% condensation temperatures (Tc) of major planet-building elements (Mg, Si, Fe, Al, Ca, Na, Ni) shift with protoplanetary disk pressure, using the ECCOplanets Gibbs free energy minimisation code over logarithmically spaced pressures from 1e-6 to 0.1 bar to recover a mean ∆Tc ≈ +350 K.

## When to use

Use when you need to determine how equilibrium condensation temperatures of rock-forming elements depend on disk pressure (e.g. testing whether element volatility defined at Solar System pressure applies to other disks), or to reproduce a ∆Tc vs. pressure sensitivity table.

## When NOT to use

- The question concerns abundance-pattern sensitivity (e.g. C/O variation from 0.1 to 0.7 giving ∆Tc ≈ −100 K) or metallicity ([M/H] −0.3 to 0.4 giving ∆Tc ≈ +100 K) rather than disk pressure — vary those parameters instead.
- You need full planet bulk compositions, devolatilisation, or feeding-zone (FZ) model outputs — use the s3planet_composition.py workflow, not a Tc-vs-pressure sweep.
- Your code cannot handle condensed phases (e.g. gas-phase-only codes like TEA) — condensation temperatures of rock-forming elements require Gibbs minimisation over solid species.

## Inputs

- Solar elemental abundance pattern (ECCOplanets stellar abundance database / Brewer+2016 data)
- ECCOplanets thermochemical database (.pkl files read via data.py)
- Simulation parameters: temperature range and resolution, molecule selection, pressure grid (log-spaced 1e-6 to 1e-1 bar)

## Outputs

- Molecular amount vs. temperature condensation curves for each pressure point (el_amounts simulation outputs)
- Per-element 50% condensation temperatures (Tc) at each pressure for Mg, Si, Fe, Al, Ca, Na, Ni
- Per-element ∆Tc relative to the 1e-6 bar baseline and mean ∆Tc across elements (results table, expected 357 ± 57 K)

## How to apply

Configure the ECCOplanets condensation simulation (simulation.py, following sample s1simulation.py) with the solar elemental abundance pattern and the code's temperature range and resolution settings. Run the Gibbs free energy minimisation model at logarithmically spaced pressures from 1e-6 to 1e-1 bar to obtain molecular amount vs. temperature curves for each pressure point. Improve each result per sample s2analyses.py by cropping the gas phase, recomputing affected sections, removing numerical glitches, and smoothing the curves. Extract the 50% condensation temperature (Tc) per element per pressure run, compute per-element ∆Tc relative to the 1e-6 bar baseline, and the mean ∆Tc across elements with pandas/numpy, writing the results table. Validation target: the mean ∆Tc must reproduce the reported 357 ± 57 K, i.e. the paper's ∆Tc ≈ +350 K for 10^-6–0.1 bar.

## Related tools

- **ECCOplanets** (Open-source Python code performing the Gibbs free energy minimisation condensation simulation across the pressure grid; provides simulation.py, s1simulation.py, and s2analyses.py workflow) — https://github.com/AninaTimmermann/ECCOplanets
- **Python** (Runtime environment for running simulations and analysis scripts)
- **pandas** (Tabulating per-element Tc and ∆Tc results across pressure runs)
- **numpy** (Numerical handling of molecular amount vs. temperature arrays and Tc extraction)
- **matplotlib** (Plotting molecular amount vs. temperature condensation curves used to inspect pressure dependence)

## Examples

```
Run s1simulation.py configured with the solar abundance pattern and p grid 1e-6 to 1e-1 bar, then per s2analyses.py: sim = Simulation('simulations/el_amounts/...pkl'); sim.improve_result(); sim.get_Tc_50_percent()
```

## Evaluation signals

- Mean ∆Tc across the tested elements must land at 357 ± 57 K when comparing the 1e-1 bar run to the 1e-6 bar baseline (paper's ∆Tc ≈ +350 K).
- Absolute Tc values for common planet-building elements must agree with literature condensation temperatures within ±5 K.
- Condensation curves should be monotonic through the condensation transition after applying the s2analyses.py improvements (cropping gas phase, recomputing a section, removing numerical glitches, smoothing).
- Tc must rise with pressure for most elements; any element showing the opposite trend should be flagged and checked for numerical glitches.
- Each pressure run must use the identical solar abundance pattern and temperature grid so ∆Tc differences are attributable to pressure alone.

## Limitations

- The code currently considers only pure minerals; solid solutions are planned but not yet included, which can affect condensation behaviour.
- The model was validated against literature Tc within ±5 K, so pressure-dependent shifts smaller than this uncertainty cannot be resolved reliably.
- Simulations on very incomplete abundance data sets (e.g. missing Al, Ca, Mg, Si, or O) may not be sensible; the first release only covers F, G, and K stars from Brewer+2016.
- Results reflect a simplified, easily extensible equilibrium model intended as an approximate alternative, not a high-fidelity thermochemical code.

## Evidence

- [abstract] Condensation temperature rises with pressure and metallicity for most elements; the tested pressure range (10^-6 - 0.1 bar) corresponds to ∆Tc ≈ +350 K: "For most elements, it rises with p and metallicity. The tested pressure range (10−6 − 0.1 bar) corresponds to ∆Tc ≈ +350 K"
- [abstract] Model agreement with literature Tc within ±5 K: "For common planet-building elements, we derive a Tc that is within ±5 K of literature values"
- [abstract] Gibbs free energy minimisation basis and ∆Tc pressure range assessment: "Our T-p-dependent chemical equilibrium model is based on a Gibbs free energy minimisation"
- [methods] Simulation parameters introduced in sample s1simulation.py: "In this sample we introduce the parameters of the condensation simulation, i.e. the temperature range and resolution, the molecule selection, and the elemental abundance pattern."
- [methods] Result improvement steps in s2analyses.py: "improving the simulation result by recomputing a section, removing numerical glitches, and smoothing the curve"
- [readme] Pure-minerals-only scope limitation: "We only consider pure minerals at the moment, but there are plans to add solid solutions at a later point."
