---
name: timestep-sweep-performance-profiling
description: Use when measuring the computational cost of the Nemesis bridge/synchronisation
  step δt_nem — i.e. sweeping dt_nem over multiple values and extracting the wall-clock
  scaling exponent against 1/dt_nem.
when_to_use_negative:
- The run uses RUN_NEMESIS = 0 (direct N-body code) — there is no bridge/synchronisation
  timestep δt_nem to profile
- The goal is validating accuracy (e.g. the von Zeipel–Lidov–Kozai secular-evolution
  test) rather than wall-clock performance
- Only a single dt_nem value is available, so no scaling exponent can be fit from
  a sweep
edam_operation: http://edamontology.org/operation_2945
tools:
- name: Nemesis
  role: Multi-scale integrator whose wall-clock time is profiled as a function of
    the bridge/synchronisation timestep dt_nem
  repo: https://github.com/ErwanH29/nemesis
- name: Ph4
  role: Cluster-optimised direct N-body integrator used as both parent and child code
    with identical internal time-step 0.1
  repo: https://github.com/ErwanH29/nemesis
- name: AMUSE
  role: Framework embedding Nemesis and providing the ph4/huayno integrator packages
    used in the test runs
  repo: https://amuse.readthedocs.io/en/latest/
- name: Python
  role: Runs the cluster test scripts (initialise_cluster.py, run_cluster.py) and
    performs the log-log linear regression of the aggregated CSV
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202558060
    title: 'Nemesis: A multi-scale, multi-physics algorithm for astrophysics'
  source_content_hash: sha256:ad7e47099d68da885eb5422deb2ba784c2f396c430beec16e803f8c86e2414ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ErwanH29/nemesis
  tools:
  - AMUSE
  - Nemesis
  - Ph4
  - Python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# timestep-sweep-performance-profiling

## Summary

Quantifies how a multi-scale integrator's wall-clock runtime depends on the synchronisation (bridge) timestep by sweeping dt_nem in Nemesis cluster runs and fitting a log-log scaling law, here expected to reproduce t_sim ∝ 1/δt_nem.

## When to use

Use when measuring the computational cost of the Nemesis bridge/synchronisation step δt_nem — i.e. sweeping dt_nem over multiple values and extracting the wall-clock scaling exponent against 1/dt_nem.

## When NOT to use

- The run uses RUN_NEMESIS = 0 (direct N-body code) — there is no bridge/synchronisation timestep δt_nem to profile
- The goal is validating accuracy (e.g. the von Zeipel–Lidov–Kozai secular-evolution test) rather than wall-clock performance
- Only a single dt_nem value is available, so no scaling exponent can be fit from a sweep

## Inputs

- Cluster initial conditions produced by tests/cluster_test/initialise_cluster.py (particle set written to the ICs/ folder)
- Sweep of bridge/synchronisation timestep values dt_nem (nominal value 500 yr)
- Nemesis run configuration: RUN_NEMESIS = 1, Ph4 parent and child integrators, code internal time-step 0.1, end time 0.1 Myr, galactic field/stellar evolution/child collisions off

## Outputs

- CSV table aggregating dt_nem versus wall-clock runtime for each run in the sweep
- Fitted log-log linear regression of wall-clock time against 1/dt_nem, yielding the scaling exponent (expected ≈ -1)

## How to apply

Generate the cluster initial conditions with tests/cluster_test/initialise_cluster.py, which writes the particle set to the ICs/ folder. For each dt_nem value in the sweep, run `python tests/cluster_test/run_cluster.py` with RUN_NEMESIS = 1, using Ph4 as both parent and child integrator with an identical code internal time-step of 0.1, an end time of 0.1 Myr (short enough to expose systematic errors while staying below the cluster's crossing time), galactic field, stellar evolution and child collisions turned off, and the bridge timestep set to each dt_nem value (nominal 500 yr). Record the wall-clock runtime of each run and aggregate dt_nem versus wall-clock time into a CSV table. Fit a log-log linear regression of wall-clock time against 1/dt_nem to extract the scaling exponent. The rationale is that Nemesis synchronises children and parents through a second-order kick-drift-kick scheme, so each bridge step costs correction kicks whose count grows as dt_nem shrinks.

## Related tools

- **Nemesis** (Multi-scale integrator whose wall-clock time is profiled as a function of the bridge/synchronisation timestep dt_nem) — https://github.com/ErwanH29/nemesis
- **Ph4** (Cluster-optimised direct N-body integrator used as both parent and child code with identical internal time-step 0.1) — https://github.com/ErwanH29/nemesis
- **AMUSE** (Framework embedding Nemesis and providing the ph4/huayno integrator packages used in the test runs) — https://amuse.readthedocs.io/en/latest/
- **Python** (Runs the cluster test scripts (initialise_cluster.py, run_cluster.py) and performs the log-log linear regression of the aggregated CSV)

## Examples

```
RUN_NEMESIS = 1; python tests/cluster_test/run_cluster.py  # sweep bridge/dt_nem values (nominal 500 yr), log wall-clock time per run
```

## Evaluation signals

- Fitted log-log exponent of wall-clock time versus 1/dt_nem is approximately -1, consistent with t_sim ∝ 1/δt_nem
- CSV contains one dt_nem versus wall-clock runtime pair for every value in the sweep, including the nominal 500 yr bridge time
- Wall-clock runtime decreases monotonically as dt_nem increases across the sweep
- Run configuration matches the cluster-test recipe: Ph4 as parent and child, identical internal time-step 0.1, end time 0.1 Myr, galactic field/stellar evolution/child collisions off

## Limitations

- The 1/δt_nem scaling holds only roughly ('scales roughly as'), so small deviations of the fitted exponent from -1 are expected
- Nemesis must be run from the same Python environment used to compile the C++ correction-kick extension, or runs will fail
- Runs can be resumed automatically, but changing simulation parameters (e.g. dt_nem) when resuming only prints a warning while the simulation proceeds — sweep points must use fresh or carefully controlled runs
- Child subsystems communicate via sockets rather than MPI; suspending workers mid-communication with many children can crash runs, adding noise to wall-clock measurements

## Evidence

- [abstract] Wall-clock time scales roughly as t_sim proportional to 1/δt_nem: "the wall-clock time scales roughly as t sim ∝ 1/ δt nem , where δt nem represents the time synchronisation between the global and local scales"
- [readme] Cluster test run command with RUN_NEMESIS flag: "To run Nemesis```: `python tests/cluster_test/run_cluster.py` with the flag `RUN_NEMESIS = 1`"
- [readme] Choice of end time for the cluster performance test: "End time: 0.1 Myr. This is a short enough time to allow any systematic errors to emerge while also remaining below the cluster's crossing time."
- [readme] Identical internal time-step requirement across integrators: "Code parameters (i.e, internal time-step) are identical between parent, child and direct N-body integrator."
- [intro] Kick-drift-kick synchronisation underlying the bridge cost: "Synchronisation between scales (children and parents) occurs through a second-order kick-drift-kick scheme."
