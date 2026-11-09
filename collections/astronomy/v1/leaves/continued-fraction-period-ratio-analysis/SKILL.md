---
name: continued-fraction-period-ratio-analysis
description: Use when two or more bodies follow Keplerian orbits around a central
  mass with intersecting orbits, and you need analytic soonest-collision times or
  a fast collision-detection schedule rather than brute-force time stepping.
when_to_use_negative:
- The system requires full gravitational N-body interaction treatment — the method
  treats gravitational interactions only as gravitational scattering or secular perturbation,
  at the cost of reducing the time-step or accuracy
- Orbits are large and dense such that collisions dominate the dynamics (e.g. N >=
  10^8 for Solar-System-like systems); large time jumps pay off only when collisions
  are rare
- Bodies are on non-intersecting Kepler orbits, so no lattice/parallel-lines collision
  geometry exists
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0621
tools:
- name: C++
  role: Main language in which the collision-jump simulations were implemented and
    run
  repo: https://github.com/Dylan-Aliberti/Kepler-Collisions
- name: Jupyter Notebook
  role: Post-processing of simulation data to make plots and calculate collision-time
    statistics
  repo: https://github.com/Dylan-Aliberti/Kepler-Collisions
- name: Python
  role: Backend of the Jupyter Notebook used to compute and validate collision-time
    statistics
- name: OpenGL
  role: Graphical interface to fly through the simulation space, used mainly for debugging
    the collision-jump algorithm
- name: Microsoft Visual Studio
  role: IDE in which the C++ code was developed; requires setting VC++ include/library
    directories and linking glfw3.lib and opengl32.lib
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202243754
    title: Collision detection for <i>N</i>-body Kepler systems
  source_content_hash: sha256:58e1a8df8b85186fc4aefe1ba2c0b96f617b67aaaf6ba62a10aed5d6fb671ac2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Dylan-Aliberti/Kepler-Collisions
  tools:
  - C++
  - Jupyter Notebook
  - Microsoft Visual Studio
  - OpenGL
  - Python
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# continued-fraction-period-ratio-analysis

## Summary

Analytic method that computes the time of collision between bodies on intersecting Kepler orbits by reducing the problem to finding the grid point between two parallel lines closest to the origin, solved via the continued fraction of the ratio of the orbital periods. It enables large jumps in time from collision to collision instead of small-step integration.

## When to use

Use when two or more bodies follow Keplerian orbits around a central mass with intersecting orbits, and you need analytic soonest-collision times or a fast collision-detection schedule rather than brute-force time stepping.

## When NOT to use

- The system requires full gravitational N-body interaction treatment — the method treats gravitational interactions only as gravitational scattering or secular perturbation, at the cost of reducing the time-step or accuracy
- Orbits are large and dense such that collisions dominate the dynamics (e.g. N >= 10^8 for Solar-System-like systems); large time jumps pay off only when collisions are rare
- Bodies are on non-intersecting Kepler orbits, so no lattice/parallel-lines collision geometry exists

## Inputs

- Orbital-element pairs (a, e, etc.) for two intersecting Kepler orbits around a common central mass
- Orbital periods of the two bodies (to form the period ratio for the continued-fraction expansion)
- Initial chronologically ordered list of possible collision pairs in an N-body Keplerian system

## Outputs

- Collision time per orbital-element pair (finite, strictly positive, bounded by the synodic/period-ratio interval)
- Validated collision_times.csv with one row per orbital-element pair
- Sequence of close-approach resonances from the continued fraction of the period ratio
- Plots and statistics produced in a Jupyter Notebook

## How to apply

For each pair of intersecting Kepler orbits, convert the orbital elements into the lattice/parallel-lines representation in which the collision-time problem becomes finding the grid point between two parallel lines that is closest to the origin. Compute the continued fraction expansion of the ratio of the two orbital periods to determine the sequence of close-approach resonances, then use the continued-fraction solution to jump in time to the soonest possible collision event for that pair. Within an N-body simulation, maintain a list of all possible collision pairs ordered chronologically and, at each step, process the soonest event on the list; only particles created in a collision can create new collision possibilities. Validate each computed collision time numerically — it must be finite, strictly positive, and bounded by the synodic/period-ratio interval for the pair — before assembling results. Simulations were run in C++ (with optional OpenGL/GLFW visualization for debugging in Visual Studio) and the outputs were processed in a Jupyter Notebook to make plots and calculate statistics.

## Related tools

- **C++** (Main language in which the collision-jump simulations were implemented and run) — https://github.com/Dylan-Aliberti/Kepler-Collisions
- **Jupyter Notebook** (Post-processing of simulation data to make plots and calculate collision-time statistics) — https://github.com/Dylan-Aliberti/Kepler-Collisions
- **Python** (Backend of the Jupyter Notebook used to compute and validate collision-time statistics)
- **OpenGL** (Graphical interface to fly through the simulation space, used mainly for debugging the collision-jump algorithm)
- **Microsoft Visual Studio** (IDE in which the C++ code was developed; requires setting VC++ include/library directories and linking glfw3.lib and opengl32.lib)

## Examples

```
python validate_collision_times.py --elements pairs.csv --periods periods.txt --out collision_times.csv
```

## Evaluation signals

- Each computed collision time is finite and strictly positive
- Each collision time is bounded by the synodic/period-ratio interval for the corresponding orbital-element pair
- collision_times.csv contains exactly one row per orbital-element pair
- The soonest event processed matches the chronologically ordered list of possible collision pairs, and only newly created particles add new collision possibilities
- Cross-check the continued-fraction-derived resonance sequence against a direct numerical search of the grid point closest to the origin between the two parallel lines

## Limitations

- Gravitational interactions between particles can only be treated as gravitational scattering or as a secular perturbation, at the cost of reducing the time-step or accuracy
- The algorithm beats tree codes only for specific systems such as the Solar System with N < 10^8
- Collisions must be rare for the jump-from-collision-to-collision approach to be fast
- Visual Studio project settings may need directory edits (libraries/include, libraries/lib) and linking of glfw3.lib and opengl32.lib before the code runs

## Evidence

- [abstract] equivalence of collision-time and closest grid point: "We find that the collision-time problem is equivalent to finding the grid point between two parallel lines that is closest to the origin."
- [abstract] continued fraction basis: "The solution is based on the continued fraction of the ratio of orbital periods."
- [abstract] chronological collision-pair list: "We outline an algorithm that maintains a list of possible collision pairs ordered chronologically."
- [readme] language and processing stack: "The main code is written in C++, where the simulations were done.
The data from the simulations was then processed in a Jupyter Notebook (which runs on Python), to make plots and/or calculate certain"
- [readme] algorithm description: "The algorithm simulates an N-body Keplerian system. It maintains a list of all collision possibilities, and jumps from collision to collision."
- [abstract] scattering limitation: "However, the gravitational interactions between particles can only be treated as gravitational scattering or as a secular perturbation, at the cost of"
