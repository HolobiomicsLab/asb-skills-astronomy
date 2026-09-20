---
name: numerical-glitch-removal-curve-smoothing
description: Use when a Gibbs free energy minimisation condensation simulation (e.g.
  ECCOplanets) produces molecule amount-vs-temperature curves containing numerical
  glitches or unphysical gas-phase artefacts that must be cleaned before extracting
  50% condensation temperatures (Tc).
when_to_use_negative:
- The simulation result is already a processed rocky planet composition table (planet
  class output) rather than a raw molecule-vs-temperature curve.
- You need to change scientific assumptions of the condensation model itself — the
  README directs such modifications to simulation.py, not to this post-processing
  step.
- Thermochemical database exploration (molecule class, s0moleculeDB.py) rather than
  simulation result cleanup is the goal.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: ECCOplanets
  role: Provides the simulation class analysis functions used to crop the gas phase,
    recompute affected sections, remove glitches, and smooth curves (samples s2analyses.py
    and s3planet_composition.py)
  repo: https://github.com/AninaTimmermann/ECCOplanets
- name: Python
  role: Language in which the improvement routines (classes.py, samples) are implemented
- name: matplotlib
  role: Used for plotting the line plots of molecule amount vs. temperature during
    analysis and improvement
provenance:
  source_task_ids:
  - task_005
  - task_008
  source_papers:
  - doi: 10.1051/0004-6361/202244850
    title: Revisiting equilibrium condensation and rocky planet compositions
  source_content_hash: sha256:b32525a0e360231bc335d02ad7c44c22f2c1c90ddcc5dd17d759509b8100e625
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# Numerical glitch removal and curve smoothing for condensation simulations

## Summary

Post-processing workflow from the ECCOplanets code that improves raw condensation simulation results by cropping the gas phase, recomputing affected temperature sections, removing numerical glitches, and smoothing the molecule-vs-temperature curves before extracting 50% condensation temperatures.

## When to use

Use when a Gibbs free energy minimisation condensation simulation (e.g. ECCOplanets) produces molecule amount-vs-temperature curves containing numerical glitches or unphysical gas-phase artefacts that must be cleaned before extracting 50% condensation temperatures (Tc).

## When NOT to use

- The simulation result is already a processed rocky planet composition table (planet class output) rather than a raw molecule-vs-temperature curve.
- You need to change scientific assumptions of the condensation model itself — the README directs such modifications to simulation.py, not to this post-processing step.
- Thermochemical database exploration (molecule class, s0moleculeDB.py) rather than simulation result cleanup is the goal.

## Inputs

- Raw ECCOplanets condensation simulation output (molecule amount vs. temperature curves from simulation.py, .pkl-based)
- Loaded simulation object via the simulation class in classes.py

## Outputs

- Improved (cropped, glitch-free, smoothed) molecule-vs-temperature curves
- Cleaned condensation curves suitable for 50% condensation temperature (Tc) extraction per element/molecule

## How to apply

After running a condensation simulation (simulation.py) or loading a stored one, follow the s2analyses.py sample using the simulation class in classes.py: first crop the gas phase, then recompute the affected temperature section to remove numerical glitches, and finally smooth the curve. Only after this improvement should element and molecule 50% condensation points be extracted (e.g. for Mg, Si, Fe, Al, Ca, Na, Ni). The same improvement sequence is embedded in the s3planet_composition.py pipeline, which takes you from running a simulation, over improving the result, to computing rocky planet bulk compositions and devolatilisation relative to the star. Correct application is judged by the derived Tc agreeing with literature values within ±5 K, which enables validation exercises such as reproducing ΔTc ≈ +357 ± 57 K for pressure variation from 1e-6 to 0.1 bar.

## Related tools

- **ECCOplanets** (Provides the simulation class analysis functions used to crop the gas phase, recompute affected sections, remove glitches, and smooth curves (samples s2analyses.py and s3planet_composition.py)) — https://github.com/AninaTimmermann/ECCOplanets
- **Python** (Language in which the improvement routines (classes.py, samples) are implemented)
- **matplotlib** (Used for plotting the line plots of molecule amount vs. temperature during analysis and improvement)

## Examples

```
python s2analyses.py  # load simulation, crop gas phase, recompute section, remove glitches, smooth curve
```

## Evaluation signals

- Derived 50% condensation temperatures for common planet-building elements fall within ±5 K of literature values.
- No residual numerical glitches or unphysical spikes remain in the molecule-vs-temperature curves after recompute/smoothing.
- Downstream aggregate results reproduce reported values, e.g. mean ΔTc ≈ +357 ± 57 K for the 1e-6 to 0.1 bar pressure range and ΔTc ≈ -100 K for C/O increase from 0.1 to 0.7.
- The improvement sequence is applied in the documented order: crop the gas phase, then recompute the affected section, then smooth the curve.

## Limitations

- Glitch removal and smoothing are part of the analysis samples; the raw simulation may contain numerical artefacts that must be handled per-run, and the code is designed as a simplified tool giving approximate results rather than high-fidelity thermodynamics.
- Only pure minerals are considered at the moment (solid solutions planned for a later point), which constrains what the smoothed curves can represent.

## Evidence

- [readme] improving the simulation result by recomputing a section, removing numerical glitches, and smoothing the curve: "improving the simulation result by recomputing a section, removing numerical glitches, and smoothing the curve"
- [readme] This sample takes you from running a simulation, over improving the result (cropping the gas phase and recomputing a section): "This sample takes you from running a simulation, over improving the result (cropping the gas phase and recomputing a section)"
- [abstract] For common planet-building elements, we derive a Tc that is within ±5 K of literature values: "For common planet-building elements, we derive a Tc that is within ±5 K of literature values"
- [readme] This is the natural second step, after running a simulation. You load a simulation, retrieve its basic parameters, and analyse the simulation results.: "This is the natural second step, after running a simulation. You load a simulation, retrieve its basic parameters, and analyse the simulation results."
