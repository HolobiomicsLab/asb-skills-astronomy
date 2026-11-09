---
name: event-driven-n-body-collision-scheduling
description: Use when simulating an N-body system of bodies on Keplerian orbits about
  a central mass where collisions are rare (e.g. Solar-System-like systems) and small-step
  integration would be wasteful.
when_to_use_negative:
- When gravitational interactions between particles must be treated fully (the method
  handles them only as gravitational scattering or secular perturbation, at the cost
  of reduced time-step or accuracy)
- For very large-N systems where tree codes are more efficient — the algorithm beats
  octree/k-d tree codes only for specific systems such as the Solar System with N
  < 10^8
- When collisions are frequent rather than rare; the jump-from-collision-to-collision
  approach is fast specifically because collisions are rare in systems like the Solar
  System
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: C++
  role: Language in which the main simulation code and the event-driven collision-list
    propagator are implemented
  repo: github.com/Dylan-Aliberti/Kepler-Collisions
- name: Microsoft Visual Studio
  role: Development environment used to build the C++ simulation; project and linker
    directories must be configured for the bundled libraries
- name: OpenGL
  role: Graphical interface allowing the user to 'fly' through the simulation space,
    mainly used for debugging the collision dynamics
- name: GLFW
  role: Windowing/input library linked (glfw3.lib) to support the OpenGL visualization
    of the simulation
- name: Jupyter Notebook
  role: Python environment for post-processing simulation data to make plots and calculate
    statistics such as the collision rate and average list-length change
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202243754
    title: Collision detection for <i>N</i>-body Kepler systems
  source_content_hash: sha256:589ac05ee1a60ea4b213a36c438c06713e30f010be818d730067519701d70612
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - C++
  - GLFW
  - Jupyter Notebook
  - Microsoft Visual Studio
  - OpenGL
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# event-driven-n-body-collision-scheduling

## Summary

An event-driven propagator for an N-body Keplerian system that keeps a chronologically ordered list of all possible collision pairs and jumps from collision to collision instead of taking small time steps, using the soonest-event update rule where only particles created in a collision can generate new collision possibilities.

## When to use

Use when simulating an N-body system of bodies on Keplerian orbits about a central mass where collisions are rare (e.g. Solar-System-like systems) and small-step integration would be wasteful.

## When NOT to use

- When gravitational interactions between particles must be treated fully (the method handles them only as gravitational scattering or secular perturbation, at the cost of reduced time-step or accuracy)
- For very large-N systems where tree codes are more efficient — the algorithm beats octree/k-d tree codes only for specific systems such as the Solar System with N < 10^8
- When collisions are frequent rather than rare; the jump-from-collision-to-collision approach is fast specifically because collisions are rare in systems like the Solar System

## Inputs

- N-body configuration with bodies on Keplerian orbits about a central mass (Solar-System-like system)
- Collision times for every particle pair computed via the continued-fraction grid-point solution
- Visual Studio project settings: include path (BaseFolder)/libraries/include, library directories (BaseFolder)/libraries/lib, additional dependencies glfw3.lib and opengl32.lib

## Outputs

- Chronologically ordered list of collision possibilities, updated after each event
- collision_events.csv with collision time, colliding-pair identifiers, and list length before and after re-insertion
- Statistics (collision rate, list length, average change in list length per event) and plots computed in a Jupyter Notebook

## How to apply

Load a Solar-System-like configuration with bodies on Keplerian orbits about a central mass, and compute the collision time for every particle pair using the continued-fraction grid-point solution of the collision-time problem (equivalent to finding the grid point between two parallel lines closest to the origin). Insert all pairs into a single list ordered chronologically by collision time. Iteratively take the soonest event from the list, resolve that collision, drop stale entries, and re-insert collision possibilities computed only for the particles created in that event — everything not touched by the collision keeps its already-computed collision time. Log each event (collision time, colliding-pair identifiers, list length before and after re-insertion) to collision_events.csv, then verify monotonic increase of logged collision times and estimate the collision rate, list length, and average change in list length per event to gauge efficiency. Simulations run in C++ (Microsoft Visual Studio with glfw3.lib and opengl32.lib linked), while post-processing of statistics and plots is done in a Jupyter Notebook (Python).

## Related tools

- **C++** (Language in which the main simulation code and the event-driven collision-list propagator are implemented) — github.com/Dylan-Aliberti/Kepler-Collisions
- **Microsoft Visual Studio** (Development environment used to build the C++ simulation; project and linker directories must be configured for the bundled libraries)
- **OpenGL** (Graphical interface allowing the user to 'fly' through the simulation space, mainly used for debugging the collision dynamics)
- **GLFW** (Windowing/input library linked (glfw3.lib) to support the OpenGL visualization of the simulation)
- **Jupyter Notebook** (Python environment for post-processing simulation data to make plots and calculate statistics such as the collision rate and average list-length change)

## Examples

```
Compute collision times for all pairs, build the chronological list, then log each resolved event to collision_events.csv: events.append((t_collision, pair_id, list_len_before, list_len_after))
```

## Evaluation signals

- Logged collision times in collision_events.csv increase monotonically, since the simulation always processes the soonest event and jumps forward in time
- After each event, new entries are re-inserted only for the particles created in that collision; stale entries for destroyed particles are dropped, so the list never contains pairs involving removed particles
- The estimated collision rate, list length, and average change in list length at an event are consistent with a system where collisions are rare (list length stays manageable)
- The C++ build succeeds with the configured include/library paths and the two additional linker dependencies glfw3.lib and opengl32.lib
- Visual inspection via the OpenGL interface (flying through the simulation space) shows the expected orbital geometry and collision behavior

## Limitations

- Gravitational interactions between particles can only be treated as gravitational scattering or as a secular perturbation, at the cost of reducing the time-step or accuracy
- The method is fast only for systems where collisions are rare; its advantage over tree codes is limited to specific systems such as the Solar System with N < 10^8
- The list of collision possibilities must be maintained and updated correctly — only particles created in a collision can cause new collision possibilities, and mishandling this update breaks the chronological ordering

## Evidence

- [abstract] chronological event list: "We outline an algorithm that maintains a list of possible collision pairs ordered chronologically."
- [abstract] soonest-event update rule: "At each step (the soonest event on the list), only the particles created in the collision can cause new collision possibilities."
- [abstract] continued-fraction solution: "The solution is based on the continued fraction of the ratio of orbital periods."
- [abstract] efficiency metrics: "We estimate the collision rate, the length of the list, and the average change in this length at an event, and study the efficiency of the method used."
- [readme] rare-collision rationale: "It maintains a list of all collision possibilities, and jumps from collision to collision. The idea is that for a lot of systems in astronomy, for example the Solar System, collisions are rare"
- [abstract] tree-code comparison: "Due to the large jumps in time, the algorithm can beat tree codes (octree and k-d tree codes can efficiently detect collisions) for specific systems such as the Solar System with N < 108."
- [readme] post-processing: "The data from the simulations was then processed in a Jupyter Notebook (which runs on Python), to make plots and/or calculate certain statistics."
