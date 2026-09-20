---
name: trust-region-optimisation-warm-starting
description: Use when minimising a bounded, constrained non-linear function across
  a sequence of closely related problems (e.g. Gibbs free energy at successive temperature
  steps) and each new problem lies near the previous solution, so warm starting accelerates
  convergence.
when_to_use_negative:
- The problem is gas-phase equilibrium only with no condensates — the article notes
  TEA already handles such simulations and ECCOplanets is designed around pure-condensate
  species
- Solid-solution minerals are required — the README states 'We only consider pure
  minerals at the moment, but there are plans to add solid solutions at a later point'
- The optimisation problem is a single standalone solve with no neighbouring solutions
  to warm start from; the benefit here comes specifically from the sequential temperature
  stepping
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: ECCOplanets
  role: Provides simulation.py, which contains the Gibbs minimisation formulation
    and the trust-region optimisation procedure parameters
  repo: https://github.com/AninaTimmermann/ECCOplanets
- name: Python
  role: Implementation language with pandas, numpy, matplotlib used to run and analyse
    the simulation
  repo: https://github.com/AninaTimmermann/ECCOplanets
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202244850
    title: Revisiting equilibrium condensation and rocky planet compositions
  source_content_hash: sha256:b42249c176d7ea67489ce1a7eae4a40b4b9bd6dd8475705bf9d4074f6760fbad
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# trust-region-optimisation-warm-starting

## Summary

Solves a sequence of bounded, constrained non-linear convex minimisation problems (Gibbs free energy over gas and condensate species) with a trust-region method, warm-starting each temperature step from the previous step's solution — the numerical backbone of ECCOplanets' condensation simulation.

## When to use

Use when minimising a bounded, constrained non-linear function across a sequence of closely related problems (e.g. Gibbs free energy at successive temperature steps) and each new problem lies near the previous solution, so warm starting accelerates convergence.

## When NOT to use

- The problem is gas-phase equilibrium only with no condensates — the article notes TEA already handles such simulations and ECCOplanets is designed around pure-condensate species
- Solid-solution minerals are required — the README states 'We only consider pure minerals at the moment, but there are plans to add solid solutions at a later point'
- The optimisation problem is a single standalone solve with no neighbouring solutions to warm start from; the benefit here comes specifically from the sequential temperature stepping

## Inputs

- Thermochemical database files (.pkl) loaded via data.py providing molecule Gibbs energies (tabulated and Shomate-fitted)
- Stellar elemental abundance pattern (e.g. from the Brewer+2016 database)
- Simulation configuration: temperature range and resolution, molecule/species selection
- Solution vector (relative molar amounts) from the previous temperature step as warm start

## Outputs

- Relative molar amounts of each gas and solid species at each temperature step, saved as condensation_simulation.pkl
- Per-element condensation temperatures (Tc) extractable by the simulation class in classes.py

## How to apply

At each temperature step, formulate the constrained convex Gibbs free energy minimisation over the selected gas-phase and pure-condensate species (Eqs. 15-18), supplying the analytic gradient and Hessian plus non-negativity and number-balance constraints. Solve with a trust-region optimiser, chosen for its stability on bounded and constrained non-linear problems, and pass the solution from the previous temperature step as the initial guess. The mathematical formulation and optimisation parameters live in simulation.py; the previous-step warm start keeps the solver on the physical branch as species condense out. Store the resulting relative molar amounts of every gas and solid species at each temperature step, then inspect for numerical glitches and recompute or smooth affected sections if needed.

## Related tools

- **ECCOplanets** (Provides simulation.py, which contains the Gibbs minimisation formulation and the trust-region optimisation procedure parameters) — https://github.com/AninaTimmermann/ECCOplanets
- **Python** (Implementation language with pandas, numpy, matplotlib used to run and analyse the simulation) — https://github.com/AninaTimmermann/ECCOplanets

## Examples

```
python samples/s1simulation.py  # configure T-range, molecule selection, abundances; simulation.py runs the trust-region solve step by step
```

## Evaluation signals

- Derived condensation temperatures for common planet-building elements must fall within ±5 K of published literature values
- Solution vectors at every temperature step must satisfy non-negativity and elemental number-balance constraints
- Inspection of molar-amount curves should show smooth condensation transitions; numerical glitches warrant recomputing a section and smoothing the curve
- Per-step convergence should be fast because the previous temperature step's solution is used as the initial guess
- Output must round-trip: simulation class in classes.py can reload condensation_simulation.pkl and retrieve its basic parameters

## Limitations

- Only pure minerals are considered; solid solutions are planned but not implemented
- Simulations on very incomplete abundance datasets may be insensible, e.g. if Al, Ca, Mg, Si, or O data is not reported
- First release only includes F, G, and K star abundances from the Brewer+2016 database; other stellar types need added data with uncertainty considered
- Numerical glitches can appear in the solved curves and must be removed by recomputing a section and smoothing

## Evidence

- [abstract] Gibbs minimisation claim: "Our T-p-dependent chemical equilibrium model is based on a Gibbs free energy minimisation."
- [readme] simulation.py role: "this is the backbone of our condensation simulation. It contains the mathematical and thermochemical formulation of our code and the parameters of the optimisation procedure."
- [readme] simulation parameters: "In this sample we introduce the parameters of the condensation simulation, i.e. the temperature range and resolution, the molecule selection, and the elemental abundance pattern."
- [abstract] validation: "For common planet-building elements, we derive a Tc that is within ±5 K of literature values"
- [methods] improving results: "improving the simulation result by recomputing a section, removing numerical glitches, and smoothing the curve"
