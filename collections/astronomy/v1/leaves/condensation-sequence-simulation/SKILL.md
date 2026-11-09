---
name: condensation-sequence-simulation
description: Use when you need equilibrium condensation sequences or 50% condensation
  temperatures (Tc) of rock-forming elements (Al, Ti, Ca, Mg, Fe, Si) for a given
  stellar abundance pattern and disk pressure, e.g. reproducing Table 2 benchmarks
  at p = 1e-4 bar.
when_to_use_negative:
- Only gas-phase chemistry is needed — the TEA code is cited as an open-source alternative
  but ECCOplanets is built for condensation with pure-condensate species
- Solid-solution minerals are required — the first release only considers pure minerals
- Simulations on very incomplete abundance data sets (e.g. if Al, Ca, Mg, Si, or O
  data is not reported) may be insensible
tools:
- name: ECCOplanets
  role: 'Core open-source Python code: simulation.py formulates the Gibbs free energy
    minimisation and trust-region optimisation; data.py loads .pkl databases; samples
    s1simulation.py/s2analyses.py configure and analyse the run'
  repo: github.com/AninaTimmermann/ECCOplanets
- name: pandas
  role: Loads the .pkl thermochemical and stellar abundance databases and stores per-temperature-step
    molar amounts
- name: numpy
  role: Numerical backbone of the optimisation procedure and equilibrium computation
- name: matplotlib
  role: Produces line plots of molecule/element abundances as a function of temperature
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202244850
    title: Revisiting equilibrium condensation and rocky planet compositions
  source_content_hash: sha256:ff6371a2e2a94bd2c51883581a7d75805ee986dc335619f496e30db443c8eab0
schema_version: 0.2.0
metadata:
  license_tier: open
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
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# condensation-sequence-simulation

## Summary

Simulates the T-p-dependent equilibrium condensation sequence of a protoplanetary disk by Gibbs free energy minimisation with a trust-region solver, using ECCOplanets to compute element and molecule abundances as a function of temperature and derive 50% condensation temperatures of planet-building elements.

## When to use

Use when you need equilibrium condensation sequences or 50% condensation temperatures (Tc) of rock-forming elements (Al, Ti, Ca, Mg, Fe, Si) for a given stellar abundance pattern and disk pressure, e.g. reproducing Table 2 benchmarks at p = 1e-4 bar.

## When NOT to use

- Only gas-phase chemistry is needed — the TEA code is cited as an open-source alternative but ECCOplanets is built for condensation with pure-condensate species
- Solid-solution minerals are required — the first release only considers pure minerals
- Simulations on very incomplete abundance data sets (e.g. if Al, Ca, Mg, Si, or O data is not reported) may be insensible

## Inputs

- Thermochemical .pkl databases from the ECCOplanets data folder
- Stellar elemental abundance pattern (e.g. Lodders2003a solar abundances; F, G, K stars from the Brewer+2016 database)
- Simulation parameters: temperature range and resolution, molecule selection, total disk pressure (e.g. 1e-4 bar)

## Outputs

- condensation_simulation.pkl: relative molar amounts of each gas and solid species at each temperature step
- 50% condensation temperatures of elements (e.g. Al 1643 K, Ti 1575 K, Ca 1512 K, Mg 1331 K, Fe 1329 K, Si 1305 K at p = 1e-4 bar)
- Abundance-vs-temperature plots (pandas/numpy/matplotlib)

## How to apply

Load the thermochemical and stellar abundance .pkl databases via data.py and configure a simulation instance in s1simulation.py, setting the temperature range (2000 K down to 300 K), temperature resolution, molecule selection (default: most common chemical species), and the elemental abundance pattern (e.g. Lodders2003a solar abundances). At each temperature step, formulation of the constrained convex Gibbs free energy minimisation problem over selected gas-phase and pure-condensate species (Eqs. 15-18) is solved with a trust-region optimisation — chosen for stability on bounded, constrained non-linear problems — supplying non-negativity and number-balance constraints plus analytic gradient and Hessian; the previous temperature step's solution is the initial guess for the next step (simulation.py). Run the simulation, which writes element and molecule abundances as a function of temperature to the simulations folder (condensation_simulation.pkl). Then in s2analyses.py, load the result and improve it by cropping the gas phase, recomputing a section, removing numerical glitches, and smoothing the abundance curves before extracting each element's 50% condensation temperature (the temperature at which half of its total abundance has condensed) via the simulation class analysis functions.

## Related tools

- **ECCOplanets** (Core open-source Python code: simulation.py formulates the Gibbs free energy minimisation and trust-region optimisation; data.py loads .pkl databases; samples s1simulation.py/s2analyses.py configure and analyse the run) — github.com/AninaTimmermann/ECCOplanets
- **pandas** (Loads the .pkl thermochemical and stellar abundance databases and stores per-temperature-step molar amounts)
- **numpy** (Numerical backbone of the optimisation procedure and equilibrium computation)
- **matplotlib** (Produces line plots of molecule/element abundances as a function of temperature)

## Examples

```
python samples/s1simulation.py  # configure and run condensation simulation (2000 K -> 300 K, p = 1e-4 bar), then analyse in s2analyses.py
```

## Evaluation signals

- Derived 50% condensation temperatures must match the benchmark values (Al 1643 K, Ti 1575 K, Ca 1512 K, Fe 1329 K, Mg 1331 K, Si 1305 K at p = 1e-4 bar) and fall within ±5 K of published literature values
- Average discrepancy from mean literature condensation temperatures should be below 5 K; largest deviation below 15 K
- Output CSV must contain all six major rock-forming elements (Al, Ti, Ca, Mg, Fe, Si)
- Each temperature step's optimisation must converge under non-negativity and number-balance constraints; numerical glitches in abundance curves should be removed before extracting Tc

## Limitations

- The first release only includes abundance data of F, G, and K stars from the Brewer+2016 database; other stellar types require care with uncertainty
- Only pure minerals are considered at the moment; solid solutions are planned for later
- The code is a simplified starting point intended to give an approximate idea of planetary composition variety, not a high-fidelity model

## Evidence

- [abstract] trust-region solver: "Our T-p-dependent chemical equilibrium model is based on a Gibbs free energy minimisation."
- [results] optimisation stability: [evidence span withheld — anchor status: not_found]
- [methods] result improvement: "improving the simulation result by recomputing a section, removing numerical glitches, and smoothing the curve"
- [abstract] validation: "For common planet-building elements, we derive a Tc that is within ±5 K of literature values"
- [readme] code structure: "simulation.py
  this is the backbone of our condensation simulation. It contains the mathematical and thermochemical formulation of our code and the parameters of the
  optimisation procedure."
