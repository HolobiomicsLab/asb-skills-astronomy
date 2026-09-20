---
name: wall-clock-timing-analysis
description: Use when measuring how the wall-clock time of Nemesis (AMUSE/Ph4 parent,
  child integrator) simulations scales with the number of planetary systems or child
  subsystems, and how the number of available cores affects that scaling.
when_to_use_negative:
- The goal is validation of dynamics (energy conservation or secular evolution such
  as the von Zeipel–Lidov–Kozai test) rather than computational scaling — use the
  ZKL or cluster test workflows instead.
- The number of available cores is below the number of child systems, so the invariance
  regime does not apply and timing reflects oversubscription rather than the skill's
  target behaviour.
- The system is a single hierarchical system without multiple independent child subsystems
  to vary in count.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Nemesis
  role: Multi-scale AMUSE framework whose simulation runs are timed to build the scaling
    table
  repo: https://github.com/ErwanH29/nemesis
- name: Ph4
  role: Cluster-optimised parent integrator launched with sufficient workers during
    the timing runs
- name: AMUSE
  role: Framework supplying the ph4/huayno integrator packages required to run the
    timed simulations
  repo: https://amuse.readthedocs.io/en/latest/
- name: Python >= 3.10
  role: Runtime for particle_initialiser.py, main.py, and assembly of scaling_timing_table.csv
- name: OpenMP
  role: Threading support required for the parallel execution whose core count governs
    the scaling regime
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202558060
    title: 'Nemesis: A multi-scale, multi-physics algorithm for astrophysics'
  source_content_hash: sha256:4eafedecd66fa82fd6114beb6462761529318606dc7bb21d4f29dc26a4146c4a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ErwanH29/nemesis
  tools:
  - AMUSE
  - Nemesis
  - OpenMP
  - Ph4
  - Python >= 3.10
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# wall-clock-timing-analysis

## Summary

Benchmarks Nemesis simulations by running the same cluster-with-planetary-systems setup once per varied number of child systems and assembling wall-clock times into a scaling table, testing the paper's claim that runtime is invariant while available cores exceed the number of systems and grows at worst linearly beyond that.

## When to use

Use when measuring how the wall-clock time of Nemesis (AMUSE/Ph4 parent, child integrator) simulations scales with the number of planetary systems or child subsystems, and how the number of available cores affects that scaling.

## When NOT to use

- The goal is validation of dynamics (energy conservation or secular evolution such as the von Zeipel–Lidov–Kozai test) rather than computational scaling — use the ZKL or cluster test workflows instead.
- The number of available cores is below the number of child systems, so the invariance regime does not apply and timing reflects oversubscription rather than the skill's target behaviour.
- The system is a single hierarchical system without multiple independent child subsystems to vary in count.

## Inputs

- Initial-condition particle set with several planetary systems (saved in ICs/ by examples/basic_cluster/particle_initialiser.py), children carrying a syst_id attribute > 0
- Command-line simulation parameters: --tend (e.g. 1Myr) and --dtbridge (e.g. 100yr)
- Core/worker configuration for the parent and child integrators set in nemesis._sub_worker

## Outputs

- scaling_timing_table.csv: wall-clock time versus number of planetary systems
- Per-run runtime diagnostics from the data/sim_stats/ directory

## How to apply

Generate initial conditions for a cluster containing several planetary systems with examples/basic_cluster/particle_initialiser.py, which saves the particle set in the ICs/ folder. Configure the number of child workers in nemesis._sub_worker so that the available cores exceed the number of child systems, ensuring both the parent (Ph4) and child integrators get sufficient workers. Run the simulation once per varied number of planetary systems via python main.py with chosen --tend and --dtbridge arguments, collecting runtime diagnostics from the automatically created data/sim_stats/ directory. Assemble wall-clock time versus number of planetary systems into a single timing table (scaling_timing_table.csv). The validation criterion is that wall-clock time stays flat across the varied system counts while cores exceed the system count, matching the paper's scalability finding.

## Related tools

- **Nemesis** (Multi-scale AMUSE framework whose simulation runs are timed to build the scaling table) — https://github.com/ErwanH29/nemesis
- **Ph4** (Cluster-optimised parent integrator launched with sufficient workers during the timing runs)
- **AMUSE** (Framework supplying the ph4/huayno integrator packages required to run the timed simulations) — https://amuse.readthedocs.io/en/latest/
- **Python >= 3.10** (Runtime for particle_initialiser.py, main.py, and assembly of scaling_timing_table.csv)
- **OpenMP** (Threading support required for the parallel execution whose core count governs the scaling regime)

## Examples

```
python main.py --tend=1Myr --dtbridge=100yr
```

## Evaluation signals

- scaling_timing_table.csv shows wall-clock time remaining unchanged as the number of planetary systems is varied, as long as the number of available cores exceeds the number of systems.
- Beyond the core-count threshold, timing increases at worst linearly with the number of excess systems — deviations stronger than linear indicate a misconfiguration.
- Each run produced a data/sim_stats/ directory with runtime diagnostics and a data/simulation_snapshot/ HDF5 output, confirming runs completed.
- Number of child workers configured in nemesis._sub_worker matches (exceeds) the number of child systems for each run, and any worker change is reflected in _worker_list.

## Limitations

- When cores are fewer than the number of planetary systems, wall-clock time is no longer invariant and the flat-scaling validation does not hold.
- Nemesis uses sockets instead of MPI for child subsystems; interrupting or suspending workers mid-communication may lead to crashes, especially with a large number of children, which can corrupt timing runs.
- Simulations must be run from the same Python environment used to compile the C++ correction-kick extension, otherwise timing runs will not execute correctly.

## Evidence

- [abstract] Wall-clock time remains unchanged with the number of planetary systems as long as available cores exceed the number of systems; beyond that, computational time increases at worst linearly with the number of excess systems: "the wall-clock time remains unchanged as long as the number of available cores exceeds the number of systems. Beyond this, we find that, at worst, the computational time increases linearly with the"
- [abstract] Wall-clock time scales roughly as t_sim proportional to 1/δt_nem, where δt_nem is the time synchronisation between global and local scales: "the wall-clock time scales roughly as t sim ∝ 1/ δt nem , where δt nem represents the time synchronisation between the global and local scales"
- [methods] Sub_worker configures the number of child workers and child integrator used in the scaling runs: "- `_sub_worker()`: Number of child workers and child integrator."
- [methods] Running the simulation via main.py with command-line arguments such as --tend and --dtbridge: "```python main.py``` If, instead, you wish to simulate your system for 1 Myr with a bridge time step of 100 yr: ```python main.py --tend=1Myr --dtbridge=100yr```"
- [readme] Output directories including sim_stats/ runtime diagnostics are automatically created under data/: "At runtime, ```Nemesis``` automatically creates output directories for a given run. These are hosted under `data/`"
- [readme] Sockets rather than MPI are used for child subsystems because of hibernation/resumption cycles, relevant to parallel timing behaviour: "```Nemesis``` uses sockets instead of MPI for child subsystems. This is because ```Nemesis``` relies heavily on hibernation/resumption cycles for its child integrators."
