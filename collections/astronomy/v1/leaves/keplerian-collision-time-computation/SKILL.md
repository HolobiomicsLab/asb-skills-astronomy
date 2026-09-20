---
name: keplerian-collision-time-computation
description: Use when two bodies follow Keplerian orbits about a central mass with
  intersecting orbital paths and you need a finite, strictly positive time to their
  next collision without stepwise numerical integration.
when_to_use_negative:
- The orbits do not intersect (the reduction to the closest grid point between two
  parallel lines presumes intersecting Kepler orbits)
- Gravitational interactions between particles must be resolved dynamically rather
  than treated as gravitational scattering or secular perturbation — the article notes
  this only comes 'at the cost of reducing the time-step or accuracy'
- You need continuous trajectory integration rather than event-driven jumps between
  rare collisions
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: Python
  role: Compute continued fractions of orbital-period ratios and calculate collision
    times for test orbital-element pairs
  repo: https://github.com/Dylan-Aliberti/Kepler-Collisions
- name: Jupyter Notebook
  role: Assemble and validate the computed collision times into tables/CSV outputs
    and calculate statistics
  repo: https://github.com/Dylan-Aliberti/Kepler-Collisions
- name: C++
  role: Runs the simulations whose collision-time computations feed the event-driven
    propagator
  repo: https://github.com/Dylan-Aliberti/Kepler-Collisions
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202243754
    title: Collision detection for <i>N</i>-body Kepler systems
  source_content_hash: sha256:097df2f01880baca548064f8fc842575189a9153bf0602c81015d91f5d5f2032
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Dylan-Aliberti/Kepler-Collisions
  tools:
  - C++
  - GLFW
  - Jupyter Notebook
  - Microsoft Visual Studio
  - OpenGL
  - Python
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# keplerian-collision-time-computation

## Summary

Computes the collision time between two bodies on intersecting Keplerian orbits analytically, by recasting the problem as finding the grid point between two parallel lines closest to the origin and solving it via the continued fraction of the ratio of orbital periods.

## When to use

Use when two bodies follow Keplerian orbits about a central mass with intersecting orbital paths and you need a finite, strictly positive time to their next collision without stepwise numerical integration.

## When NOT to use

- The orbits do not intersect (the reduction to the closest grid point between two parallel lines presumes intersecting Kepler orbits)
- Gravitational interactions between particles must be resolved dynamically rather than treated as gravitational scattering or secular perturbation — the article notes this only comes 'at the cost of reducing the time-step or accuracy'
- You need continuous trajectory integration rather than event-driven jumps between rare collisions

## Inputs

- Pairs of orbital elements for bodies on intersecting Keplerian orbits about a central mass
- Orbital periods of the two bodies in each pair (for the continued-fraction expansion of their ratio)

## Outputs

- Collision time for each orbital-element pair (finite, strictly positive, bounded by the synodic/period-ratio interval)
- Validated collision-time table, e.g. collision_times.csv with one row per orbital-element pair

## How to apply

For each pair of intersecting Kepler orbits, convert the orbital elements into the lattice/parallel-lines representation in which the collision-time problem reduces to finding the grid point between two parallel lines closest to the origin. Expand the ratio of the two orbital periods as a continued fraction and use that expansion to determine the sequence of close-approach resonances and the soonest grid point, which gives the collision time. Validate each result by checking that it is finite, strictly positive, and bounded by the synodic/period-ratio interval for the pair. Insert validated pair times into a chronologically ordered list so an event-driven propagator can jump from collision to collision. Assemble results into a table such as collision_times.csv with one row per orbital-element pair.

## Related tools

- **Python** (Compute continued fractions of orbital-period ratios and calculate collision times for test orbital-element pairs) — https://github.com/Dylan-Aliberti/Kepler-Collisions
- **Jupyter Notebook** (Assemble and validate the computed collision times into tables/CSV outputs and calculate statistics) — https://github.com/Dylan-Aliberti/Kepler-Collisions
- **C++** (Runs the simulations whose collision-time computations feed the event-driven propagator) — https://github.com/Dylan-Aliberti/Kepler-Collisions

## Examples

```
python compute_collision_times.py --orbits orbital_pairs.csv --output collision_times.csv
```

## Evaluation signals

- Every computed collision time is finite and strictly positive
- Each collision time is bounded by the synodic/period-ratio interval for that orbital-element pair
- collision_times.csv contains exactly one row per input orbital-element pair
- Logged collision times consumed by the propagator increase monotonically across events

## Limitations

- Gravitational interactions between particles can only be treated as gravitational scattering or as a secular perturbation, at the cost of reducing the time-step or accuracy
- The method targets systems where collisions are rare (e.g. the Solar System); its efficiency claim via large time jumps is specific to such systems (N < 10^8 for Solar-System-like configurations to beat tree codes)

## Evidence

- [abstract] collision-time problem equivalent to closest grid point: "We find that the collision-time problem is equivalent to finding the grid point between two parallel lines that is closest to the origin."
- [abstract] continued fraction of period ratio: "The solution is based on the continued fraction of the ratio of orbital periods."
- [abstract] chronological list algorithm: "We outline an algorithm that maintains a list of possible collision pairs ordered chronologically."
- [abstract] scattering/secular perturbation caveat: "the gravitational interactions between particles can only be treated as gravitational scattering or as a secular perturbation, at the cost of"
- [readme] Jupyter/Python processing: "The data from the simulations was then processed in a Jupyter Notebook (which runs on Python), to make plots and/or calculate certain statistics."
