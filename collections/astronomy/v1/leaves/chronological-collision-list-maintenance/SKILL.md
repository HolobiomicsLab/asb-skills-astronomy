---
name: chronological-collision-list-maintenance
description: Use when simulating a Keplerian N-body system (e.g. a Solar-System-like
  configuration with bodies on Keplerian orbits about a central mass) where collisions
  are rare, and collision times per pair can be computed analytically via the continued-fraction
  grid-point solution.
when_to_use_negative:
- When gravitational interactions between particles must be treated beyond gravitational
  scattering or secular perturbation — the method treats interactions only at the
  cost of reducing the time-step or accuracy
- When collisions are frequent (the collision-to-collision jumping approach is a fast
  method specifically for systems like the Solar System where collisions are rare)
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: C++
  role: Language in which the event-driven simulation with the chronological collision
    list is implemented and run
  repo: github.com/Dylan-Aliberti/Kepler-Collisions
- name: Microsoft Visual Studio
  role: IDE used to develop and build the simulation; requires setting include/library
    directories and linker dependencies (glfw3.lib, opengl32.lib)
- name: GLFW
  role: Library linked (glfw3.lib) to support the OpenGL graphical interface used
    to fly through the simulation space for debugging
- name: OpenGL
  role: Graphics library (opengl32.lib) providing the visual debugging interface for
    observing the simulation
- name: Python
  role: Used via Jupyter Notebook to process simulation output and compute statistics
    such as collision rate and list-length changes
- name: Jupyter Notebook
  role: Environment for post-processing simulation data into plots and statistics
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202243754
    title: Collision detection for <i>N</i>-body Kepler systems
  source_content_hash: sha256:c5c3e16b238be41c67d9620ab2da05f9b0872a25b075eb41af329976fa2e7fe5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - C++
  - GLFW
  - Jupyter Notebook
  - Microsoft Visual Studio
  - OpenGL
  - Python
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# chronological-collision-list-maintenance

## Summary

Event-driven propagation technique for N-body Keplerian systems: keep a chronologically ordered list of all collision-pair possibilities, process the soonest event, and re-insert only the collision possibilities involving particles created in that collision, so the simulation jumps from collision to collision.

## When to use

Use when simulating a Keplerian N-body system (e.g. a Solar-System-like configuration with bodies on Keplerian orbits about a central mass) where collisions are rare, and collision times per pair can be computed analytically via the continued-fraction grid-point solution.

## When NOT to use

- When gravitational interactions between particles must be treated beyond gravitational scattering or secular perturbation — the method treats interactions only at the cost of reducing the time-step or accuracy
- When collisions are frequent (the collision-to-collision jumping approach is a fast method specifically for systems like the Solar System where collisions are rare)

## Inputs

- N-body Keplerian configuration file with bodies on Keplerian orbits about a central mass
- Per-pair collision times computed from the continued-fraction grid-point solution
- Visual Studio project settings for the C++ simulation code (include path (BaseFolder)/libraries/include, library directories (BaseFolder)/libraries/lib, additional dependencies glfw3.lib and opengl32.lib)

## Outputs

- Chronologically ordered list of collision possibilities (list length tracked per event)
- collision_events.csv with collision time, colliding-pair identifiers, and list length before and after re-insertion
- Summary statistics: collision rate, list length, and average change in list length per event

## How to apply

For every particle pair, compute the collision time using the continued-fraction solution of the collision-time problem (equivalent to finding the grid point between two parallel lines closest to the origin, based on the continued fraction of the ratio of orbital periods) and insert all pairs into a list ordered chronologically. Iteratively take the soonest event from the list, resolve that collision, drop stale entries, and re-insert collision possibilities computed only for the particles created in the event — this locality is what makes the update cheap. After each event, log the collision time, the colliding-pair identifiers, and the list length before and after re-insertion (e.g. to collision_events.csv), then continue to the next event. Verify that logged collision times increase monotonically, and estimate the collision rate, the length of the list, and the average change in list length per event to study method efficiency. Run the simulations in C++ (developed in Microsoft Visual Studio with GLFW/OpenGL libraries), and process output in a Jupyter Notebook for plots and statistics.

## Related tools

- **C++** (Language in which the event-driven simulation with the chronological collision list is implemented and run) — github.com/Dylan-Aliberti/Kepler-Collisions
- **Microsoft Visual Studio** (IDE used to develop and build the simulation; requires setting include/library directories and linker dependencies (glfw3.lib, opengl32.lib))
- **GLFW** (Library linked (glfw3.lib) to support the OpenGL graphical interface used to fly through the simulation space for debugging)
- **OpenGL** (Graphics library (opengl32.lib) providing the visual debugging interface for observing the simulation)
- **Python** (Used via Jupyter Notebook to process simulation output and compute statistics such as collision rate and list-length changes)
- **Jupyter Notebook** (Environment for post-processing simulation data into plots and statistics)

## Evaluation signals

- Logged collision times in collision_events.csv increase monotonically across events
- After resolving an event, new collision possibilities are inserted only for the particles created in that collision — no other pairs are re-computed
- List length before and after re-insertion is recorded per event, and the average change in list length per event is finite and consistent with a rare-collision system
- Estimated collision rate is consistent with a system where collisions are rare (e.g. a Solar-System-like configuration)
- Simulated trajectory advances by large time jumps, from collision to collision, rather than through uniform small time-steps

## Limitations

- Gravitational interactions between particles can only be treated as gravitational scattering or as a secular perturbation, at the cost of reducing the time-step or the accuracy
- The method is efficient for specific systems such as the Solar System with N < 10^8, where large time jumps let it beat tree codes; other regimes may not benefit
- Setting up the project in Visual Studio requires manually changing directories (include: (BaseFolder)/libraries/include; library: (BaseFolder)/libraries/lib) and adding glfw3.lib and opengl32.lib as linker dependencies

## Evidence

- [abstract] Maintain a list of possible collision pairs ordered chronologically: "We outline an algorithm that maintains a list of possible collision pairs ordered chronologically."
- [abstract] Only new particles generate new collision possibilities: "At each step (the soonest event on the list), only the particles created in the collision can cause new collision possibilities."
- [readme] Algorithm simulates N-body Keplerian system and jumps from collision to collision: "The algorithm simulates an N-body Keplerian system. It maintains a list of all collision possibilities, and jumps from collision to collision."
- [readme] Visual Studio linker dependencies: [evidence span withheld — anchor status: external_doc]
- [abstract] Collision-time problem as grid-point search: "We find that the collision-time problem is equivalent to finding the grid point between two parallel lines that is closest to the origin."
- [abstract] Efficiency metrics studied: "We estimate the collision rate, the length of the list, and the average change in this length at an event, and study the efficiency of the method used."
