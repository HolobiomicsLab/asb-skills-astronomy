---
name: orbital-close-approach-time-series-computation
description: Use when you need the finite, positive collision time between two bodies
  on intersecting Kepler orbits from their orbital elements, or when simulating a
  Keplerian system where collisions are rare and per-pair analytic collision times
  enable jumping from event to event.
when_to_use_negative:
- 'Interactions are not Keplerian two-body encounters: the article notes gravitational
  interactions between particles can only be treated as gravitational scattering or
  secular perturbation, at the cost of reduced time-step or accuracy'
- Systems where collisions are frequent rather than rare — the event-jumping algorithm
  is motivated by systems like the Solar System where collisions are rare
- Full force-integration between particles is required, since the propagator jumps
  collision to collision rather than integrating gravitational interactions continuously
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3399
- http://edamontology.org/topic_3318
tools:
- name: C++
  role: Runs the simulations and implements the analytic propagator that maintains
    the collision list and jumps from collision to collision
  repo: github.com/Dylan-Aliberti/Kepler-Collisions
- name: Jupyter Notebook
  role: Processes simulation output to make plots and calculate statistics on collision
    events
  repo: github.com/Dylan-Aliberti/Kepler-Collisions
- name: Python
  role: Processing/validation of simulated collision-time data inside the Jupyter
    Notebook
  repo: github.com/Dylan-Aliberti/Kepler-Collisions
- name: OpenGL
  role: Graphical interface to 'fly' through the simulation space, mainly for debugging
    collision events
  repo: github.com/Dylan-Aliberti/Kepler-Collisions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202243754
    title: Collision detection for <i>N</i>-body Kepler systems
  source_content_hash: sha256:553bfec384afe90c39c6c1056538da17a12f90c65397389159a16add2af05671
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - C++
  - Jupyter Notebook
  - OpenGL
  - Python
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3399
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# orbital-close-approach-time-series-computation

## Summary

Computes analytic collision/close-approach times between bodies on intersecting Kepler orbits by reducing the problem to finding the grid point between two parallel lines closest to the origin, solved via the continued fraction of the orbital-period ratio, enabling large time jumps in N-body Keplerian simulations.

## When to use

Use when you need the finite, positive collision time between two bodies on intersecting Kepler orbits from their orbital elements, or when simulating a Keplerian system where collisions are rare and per-pair analytic collision times enable jumping from event to event.

## When NOT to use

- Interactions are not Keplerian two-body encounters: the article notes gravitational interactions between particles can only be treated as gravitational scattering or secular perturbation, at the cost of reduced time-step or accuracy
- Systems where collisions are frequent rather than rare — the event-jumping algorithm is motivated by systems like the Solar System where collisions are rare
- Full force-integration between particles is required, since the propagator jumps collision to collision rather than integrating gravitational interactions continuously

## Inputs

- Orbital-element pairs (a, e, i, angles, period) for two bodies on intersecting Kepler orbits
- Orbital periods of the two bodies (for the period-ratio continued fraction)
- List of possible collision pairs in a Keplerian N-body system (for chronological event stepping)

## Outputs

- Computed collision time for each orbital-element pair (finite, positive, bounded by the synodic interval)
- Sequence of close-approach resonances derived from the continued fraction of the period ratio
- Validated collision-time table, e.g. collision_times.csv with one row per orbital-element pair
- Chronologically ordered collision-event list used to jump from collision to collision

## How to apply

For each pair of intersecting Kepler orbits given as orbital elements, convert the pair into the lattice/parallel-lines representation in which the collision-time problem reduces to finding the grid point between two parallel lines that is closest to the origin. Compute the continued fraction of the ratio of the two orbital periods; the expansion determines the sequence of close-approach resonances and the soonest collision event. In an N-body Keplerian system, maintain a list of possible collision pairs ordered chronologically and, at each step, jump to the soonest event on the list; only particles created in a collision need their new collision possibilities inserted. Validate each computed collision time numerically: it must be finite, strictly positive, and bounded by the synodic/period-ratio interval for that pair, then assemble the validated times into a table (e.g. collision_times.csv, one row per orbital-element pair).

## Related tools

- **C++** (Runs the simulations and implements the analytic propagator that maintains the collision list and jumps from collision to collision) — github.com/Dylan-Aliberti/Kepler-Collisions
- **Jupyter Notebook** (Processes simulation output to make plots and calculate statistics on collision events) — github.com/Dylan-Aliberti/Kepler-Collisions
- **Python** (Processing/validation of simulated collision-time data inside the Jupyter Notebook) — github.com/Dylan-Aliberti/Kepler-Collisions
- **OpenGL** (Graphical interface to 'fly' through the simulation space, mainly for debugging collision events) — github.com/Dylan-Aliberti/Kepler-Collisions

## Examples

```
python validate_collision_times.py --pairs orbital_elements.csv --output collision_times.csv
```

## Evaluation signals

- Each computed collision time is finite and strictly positive
- Each collision time is bounded by the synodic/period-ratio interval for that orbital-element pair
- The grid-point formulation agrees: the solution corresponds to the grid point between the two parallel lines closest to the origin
- The event list remains consistent: after a collision, only the newly created particles generate new collision possibilities
- Output table (e.g. collision_times.csv) contains exactly one validated row per orbital-element pair

## Limitations

- Gravitational interactions between particles can only be treated as gravitational scattering or as a secular perturbation, at the cost of reducing the time-step or the accuracy
- The method's advantage relies on collisions being rare; performance depends on the collision rate, list length, and average change in list length at each event
- The algorithm beats tree codes only for specific systems such as the Solar System with N < 10^8

## Evidence

- [abstract] collision-time equivalence to grid point: "We find that the collision-time problem is equivalent to finding the grid point between two parallel lines that is closest to the origin."
- [abstract] continued fraction basis: "The solution is based on the continued fraction of the ratio of orbital periods."
- [abstract] chronological event-jumping algorithm: "At each step (the soonest event on the list), only the particles created in the collision can cause new collision possibilities."
- [readme] implementation and tools: "The algorithm simulates an N-body Keplerian system. It maintains a list of all collision possibilities, and jumps from collision to collision."
- [abstract] performance domain: "Due to the large jumps in time, the algorithm can beat tree codes (octree and k-d tree codes can efficiently detect collisions) for specific systems such as the Solar System with N < 108."
