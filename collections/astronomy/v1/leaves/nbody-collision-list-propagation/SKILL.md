---
name: nbody-collision-list-propagation
description: Decision support for when an event-driven Kepler collision-list propagator
  can beat tree-code (octree, k-d tree) collision detection, e.g. Solar-System-like
  systems with rare collisions and N < 10^8.
when_to_use_negative:
- Collision-dense systems where the list of collision possibilities grows large —
  the approach is motivated by systems such as the Solar System where collisions are
  rare
- Simulations that need full gravitational interactions between particles treated
  beyond scattering or secular perturbation, since that comes at the cost of reducing
  the time-step or accuracy
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: C++
  role: Language of the main Kepler-Collisions propagator and simulation code whose
    runtime is benchmarked
  repo: github.com/Dylan-Aliberti/Kepler-Collisions
- name: Microsoft Visual Studio
  role: IDE used to build the C++ code and configure library/linker paths (glfw3.lib,
    opengl32.lib)
  repo: github.com/Dylan-Aliberti/Kepler-Collisions
- name: Python
  role: Processes simulation timings and computes propagator/baseline time ratios
  repo: github.com/Dylan-Aliberti/Kepler-Collisions
- name: Jupyter Notebook
  role: Environment for computing timing ratios, making plots, and calculating statistics
  repo: github.com/Dylan-Aliberti/Kepler-Collisions
- name: OpenGL
  role: Graphical interface to 'fly' through the simulation space, used mainly for
    debugging the propagator
- name: GLFW
  role: Windowing library (glfw3.lib) linked into the C++ graphical interface
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202243754
    title: Collision detection for <i>N</i>-body Kepler systems
  source_content_hash: sha256:3140c228070917834b8771b18e5c79eddabadfa7373f35ea1ed11dfcde4925a0
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
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# nbody-collision-list-propagation

## Summary

Benchmark an event-driven analytic Kepler propagator that keeps a chronologically ordered list of possible collision pairs and jumps from collision to collision, against octree and k-d tree collision-detection baselines across N-body particle counts.

## When to use

Decision support for when an event-driven Kepler collision-list propagator can beat tree-code (octree, k-d tree) collision detection, e.g. Solar-System-like systems with rare collisions and N < 10^8.

## When NOT to use

- Collision-dense systems where the list of collision possibilities grows large — the approach is motivated by systems such as the Solar System where collisions are rare
- Simulations that need full gravitational interactions between particles treated beyond scattering or secular perturbation, since that comes at the cost of reducing the time-step or accuracy

## Inputs

- Solar-System-like Keplerian N-body initial configurations spanning particle counts N < 10^8
- Built Kepler-Collisions C++ executable (Visual Studio project with GLFW/OpenGL libraries)
- Octree and k-d tree collision-detection baseline executables

## Outputs

- Wall-clock timing logs for propagator, octree, and k-d tree runs per configuration
- timing_comparison.csv with propagator_time / baseline_time ratios per baseline and per N
- Efficiency statistics (collision rate, list length, average change in list length at an event)

## How to apply

Build the Kepler-Collisions C++ code in Microsoft Visual Studio, setting Library directories to (BaseFolder)/libraries/lib and adding glfw3.lib and opengl32.lib as additional dependencies under Linker -> Input. Generate matched Solar-System-like Keplerian N-body initial configurations spanning particle counts N below 10^8. Run the analytic propagator, which maintains a chronologically ordered list of possible collision pairs and at each step jumps to the soonest event, logging wall-clock time per configuration. Run the octree and k-d tree collision-detection baselines on identical configurations, then compute the propagator/baseline time ratio in a Jupyter Notebook (Python) and write timing_comparison.csv. Validation: the ratio must fall below 1 for at least the small-N Solar-System regime (N < 10^8) claimed in the paper.

## Related tools

- **C++** (Language of the main Kepler-Collisions propagator and simulation code whose runtime is benchmarked) — github.com/Dylan-Aliberti/Kepler-Collisions
- **Microsoft Visual Studio** (IDE used to build the C++ code and configure library/linker paths (glfw3.lib, opengl32.lib)) — github.com/Dylan-Aliberti/Kepler-Collisions
- **Python** (Processes simulation timings and computes propagator/baseline time ratios) — github.com/Dylan-Aliberti/Kepler-Collisions
- **Jupyter Notebook** (Environment for computing timing ratios, making plots, and calculating statistics) — github.com/Dylan-Aliberti/Kepler-Collisions
- **OpenGL** (Graphical interface to 'fly' through the simulation space, used mainly for debugging the propagator)
- **GLFW** (Windowing library (glfw3.lib) linked into the C++ graphical interface)

## Examples

```
python -c "import pandas as pd; df=pd.read_csv('timing_comparison.csv'); assert (df[df.N<1e8].ratio<1).all()"
```

## Evaluation signals

- timing_comparison.csv shows propagator_time / baseline_time ratio < 1 for at least the small-N Solar-System regime (N < 10^8)
- Same matched initial configurations are used for propagator, octree, and k-d tree runs so ratios are comparable per N
- Collision events occur in the expected chronological order, with only particles created in a collision generating new collision possibilities
- Logged statistics on collision rate, list length, and average change in list length at an event are consistent with a rare-collision regime

## Limitations

- The efficiency win over tree codes holds only for specific systems, such as the Solar System with N < 10^8
- Gravitational interactions between particles can only be treated as gravitational scattering or secular perturbation, at the cost of reducing the time-step or accuracy
- Building in Visual Studio requires manually fixing project directory paths (libraries/include, libraries/lib) and linker dependencies

## Evidence

- [abstract] Due to large time jumps, the algorithm can beat tree codes: "Due to the large jumps in time, the algorithm can beat tree codes (octree and k-d tree codes can efficiently detect collisions) for specific systems such as the Solar System with N < 108."
- [abstract] Algorithm maintains ordered list of collision pairs: "We outline an algorithm that maintains a list of possible collision pairs ordered chronologically."
- [abstract] Only new particles create new collision possibilities: "At each step (the soonest event on the list), only the particles created in the collision can cause new collision possibilities."
- [readme] Rare-collision rationale: "The idea is that for a lot of systems in astronomy, for example the Solar System, collisions are rare, thus making this approach a fast method."
- [methods] Visual Studio linker configuration: [evidence span withheld — anchor status: external_doc]
- [intro] Data processing in Jupyter: "The data from the simulations was then processed in a Jupyter Notebook (which runs on Python), to make plots and/or calculate certain statistics."
