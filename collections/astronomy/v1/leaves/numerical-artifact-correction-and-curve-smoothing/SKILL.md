---
name: numerical-artifact-correction-and-curve-smoothing
description: Use when a Gibbs-minimisation condensation simulation produces molecule-abundance
  curves with numerical glitches or spikes, and you need clean curves before extracting
  condensation temperatures.
when_to_use_negative:
- No condensation simulation has been run yet — the filters operate on simulation
  results, not raw thermochemical database files
- The study is purely gas-phase chemical equilibrium (e.g. TEA-style simulations),
  where there is no condensation curve to smooth
- You need the raw simulation output unchanged, e.g. for diagnosing the optimisation
  procedure in simulation.py itself
tools:
- name: ECCOplanets
  role: Provides the s2analyses.py sample utilities and the simulation class in classes.py
    used to crop the gas phase, recompute sections, remove glitches and smooth curves
  repo: https://github.com/AninaTimmermann/ECCOplanets
- name: Python
  role: Runtime environment for the s2analyses.py post-processing, together with pandas,
    numpy and matplotlib
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202244850
    title: Revisiting equilibrium condensation and rocky planet compositions
  source_content_hash: sha256:e648a65b169e96fecb4fe931f3d50b49e24a2acbaa3fa9b90b3541d8b88f8547
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# Numerical artifact correction and curve smoothing

## Summary

Post-processing step of the ECCOplanets condensation workflow: after Gibbs free energy minimisation, the raw molecule-abundance vs. temperature curves are cleaned up (gas phase cropped, a section recomputed, numerical glitches removed, curves smoothed) so that 50% condensation temperatures can be extracted reliably.

## When to use

Use when a Gibbs-minimisation condensation simulation produces molecule-abundance curves with numerical glitches or spikes, and you need clean curves before extracting condensation temperatures.

## When NOT to use

- No condensation simulation has been run yet — the filters operate on simulation results, not raw thermochemical database files
- The study is purely gas-phase chemical equilibrium (e.g. TEA-style simulations), where there is no condensation curve to smooth
- You need the raw simulation output unchanged, e.g. for diagnosing the optimisation procedure in simulation.py itself

## Inputs

- Completed ECCOplanets condensation simulation object (Gibbs free energy minimisation output, e.g. produced via s1simulation.py at 1e-4 bar with Solar System abundances)
- Molecule-abundance vs. temperature curves from the simulation class in classes.py

## Outputs

- Cleaned, smoothed molecule-abundance curves (gas phase cropped, numerical glitches removed)
- Reliable 50% condensation temperatures for target elements/species, ready for comparison against Lodders2003a benchmark values in condensation_temperatures.csv

## How to apply

Load the finished simulation object and apply the s2analyses.py utilities in order: first crop the gas phase, then recompute a section of the temperature grid to remove numerical glitches, and finally smooth the molecule-abundance curves. These filter steps precede condensation-temperature extraction (the 50% condensation point of elements such as corundum, perovskite, gehlenite, spinel, anorthite, iron, forsterite, diopside, enstatite). The rationale is that raw Gibbs free energy minimisation output contains artefacts that would otherwise bias the 50% condensation point; the cleaned curves are the basis for benchmarking against Lodders2003a Condor values in condensation_temperatures.csv. If the artefacts persist, revisit the simulation parameters (temperature range/resolution, molecule selection) defined in s1simulation.py rather than over-smoothing.

## Related tools

- **ECCOplanets** (Provides the s2analyses.py sample utilities and the simulation class in classes.py used to crop the gas phase, recompute sections, remove glitches and smooth curves) — https://github.com/AninaTimmermann/ECCOplanets
- **Python** (Runtime environment for the s2analyses.py post-processing, together with pandas, numpy and matplotlib)

## Examples

```
python samples/s2analyses.py  # load simulation, crop gas phase, recompute section, remove glitches, smooth curves, extract Tc(50%)
```

## Evaluation signals

- Cleaned molecule-abundance curves show no numerical glitches or spikes across the configured temperature grid
- Derived 50% condensation temperatures of common planet-building elements fall within ±5 K of Lodders2003a Condor literature values (paper's benchmark criterion)
- Raw-vs-cleaned comparison shows deviations mostly within ±50 K on average, attributable only to the differing 50% condensation-temperature definition
- Condensation temperatures extracted after filtering agree for all target species (corundum, perovskite, gehlenite, spinel, anorthite, iron, forsterite, diopside, enstatite) written to condensation_temperatures.csv

## Limitations

- The code only considers pure minerals at the moment; solid solutions are planned for a later release, which can affect curve shapes
- Simulations on very incomplete abundance datasets (e.g. missing Al, Ca, Mg, Si, or O) may be scientifically unmeaningful and produce unreliable curves
- Over-smoothing can obscure real condensation steps; if artefacts persist the simulation parameters (temperature range and resolution, molecule selection) may need revisiting

## Evidence

- [methods] Filter steps (candidate fine-grained sub-tasks): cropping the gas phase / recomputing a section: "improving the simulation result by recomputing a section, removing numerical glitches, and smoothing the curve"
- [readme] Post-simulation analysis scope of s2analyses.py: "and improving the simulation result by recomputing a section, removing numerical glitches, and smoothing the curve"
- [abstract] Validation of the overall workflow against literature: "For common planet-building elements, we derive a Tc that is within ±5 K of literature values"
- [methods] Simulation setup context for the curves being filtered: "In this sample we introduce the parameters of the condensation simulation, i.e. the temperature range and resolution, the molecule selection, and the elemental abundance pattern."
