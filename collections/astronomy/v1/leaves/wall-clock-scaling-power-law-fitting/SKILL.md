---
name: wall-clock-scaling-power-law-fitting
description: Use when you need to validate or characterise the computational cost
  scaling of a simulation algorithm against a tunable timestep or resolution parameter,
  and a published scaling claim (e.g. an inverse power law) exists to test against.
when_to_use_negative:
- When other performance factors vary between runs (e.g. number of child systems exceeding
  available cores, or stellar evolution/galactic field left on), since these introduce
  confounds that break the clean 1/δt_nem relationship
- When the goal is to measure scaling with the number of planetary systems rather
  than the synchronisation timestep — that scaling (linear with excess systems beyond
  core count) is a separate behaviour
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Nemesis
  role: Multi-scale algorithm whose wall-clock time is benchmarked as a function of
    the synchronisation timestep δt_nem
  repo: https://github.com/ErwanH29/nemesis
- name: AMUSE
  role: Framework in which Nemesis and its integrators are embedded and run
  repo: https://amuse.readthedocs.io/en/latest/
- name: Ph4
  role: Cluster-optimised direct N-body integrator used as both parent and child integrator
    with identical code internal time-step (0.1)
  repo: https://github.com/ErwanH29/nemesis
- name: Python >= 3.10
  role: Scripting the sweep, runtime recording, CSV aggregation, and log-log regression
    fit
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202558060
    title: 'Nemesis: A multi-scale, multi-physics algorithm for astrophysics'
  source_content_hash: sha256:6dac022f8352c52eb44c332e8fa3cfc0b066dc417b43d2fb7b1753a0cc85836e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ErwanH29/nemesis
  tools:
  - AMUSE
  - Nemesis
  - Ph4
  - Python
  - Python >= 3.10
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# wall-clock-scaling-power-law-fitting

## Summary

Empirically measures how a simulation's wall-clock runtime scales with a control parameter by sweeping the parameter over multiple runs and fitting a power law via log-log linear regression. Used here to validate the Nemesis result t_sim ∝ 1/δt_nem, where δt_nem is the synchronisation timestep between the global and local scales.

## When to use

Use when you need to validate or characterise the computational cost scaling of a simulation algorithm against a tunable timestep or resolution parameter, and a published scaling claim (e.g. an inverse power law) exists to test against.

## When NOT to use

- When other performance factors vary between runs (e.g. number of child systems exceeding available cores, or stellar evolution/galactic field left on), since these introduce confounds that break the clean 1/δt_nem relationship
- When the goal is to measure scaling with the number of planetary systems rather than the synchronisation timestep — that scaling (linear with excess systems beyond core count) is a separate behaviour

## Inputs

- Cluster initial conditions particle set written to the ICs/ folder by initialise_cluster.py
- Sweep of bridge/synchronisation timestep values dt_nem (nominal 500 yr)
- Wall-clock runtime measurements recorded per run

## Outputs

- CSV table of dt_nem versus wall-clock time
- Fitted log-log regression slope (scaling exponent), expected ≈ -1

## How to apply

Set up a controlled benchmark where only the target parameter varies: for Nemesis, generate cluster initial conditions with tests/cluster_test/initialise_cluster.py, then run tests/cluster_test/run_cluster.py with RUN_NEMESIS = 1, using Ph4 as parent and child integrator with an identical code internal time-step of 0.1, end time 0.1 Myr, and galactic field, stellar evolution, and child collisions turned off so performance is not confounded. Sweep the bridge/synchronisation timestep δt_nem (nominal value 500 yr) across a range of values, recording the wall-clock runtime of each run. Aggregate δt_nem versus wall-clock time into a CSV table, then fit a linear regression of log(wall-clock time) against log(1/dt_nem) to extract the scaling exponent. The fitted exponent should be approximately -1, confirming t_sim ∝ 1/δt_nem.

## Related tools

- **Nemesis** (Multi-scale algorithm whose wall-clock time is benchmarked as a function of the synchronisation timestep δt_nem) — https://github.com/ErwanH29/nemesis
- **AMUSE** (Framework in which Nemesis and its integrators are embedded and run) — https://amuse.readthedocs.io/en/latest/
- **Ph4** (Cluster-optimised direct N-body integrator used as both parent and child integrator with identical code internal time-step (0.1)) — https://github.com/ErwanH29/nemesis
- **Python >= 3.10** (Scripting the sweep, runtime recording, CSV aggregation, and log-log regression fit)

## Examples

```
python tests/cluster_test/run_cluster.py  # RUN_NEMESIS = 1; sweep --dtbridge over values (nominal 500 yr); fit log(time) vs log(1/dt_nem)
```

## Evaluation signals

- Fitted log-log exponent of wall-clock time versus 1/dt_nem should be approximately -1, matching the reported t_sim ∝ 1/δt_nem scaling
- All runs except δt_nem must use identical settings: Ph4 for parent and child, internal time-step 0.1, end time 0.1 Myr, galactic field/stellar evolution/child collisions off
- Wall-clock times must be recorded for each dt_nem value in the sweep and tabulated in the CSV
- Regression should be performed on log-transformed data (log time vs log(1/dt_nem)), not raw values

## Limitations

- The scaling is only 'roughly' t_sim ∝ 1/δt_nem — the fitted exponent is approximate, so small deviations from -1 are expected
- Performance comparison requires galactic field, stellar evolution, and child collisions turned off so added physics does not dominate runtime
- Nemesis must be run from the same Python environment used to compile its C++ correction-kick extension, or runs will fail
- Resuming runs with changed simulation parameters only prints a warning; the simulation proceeds, so swept runs must be started cleanly to avoid contaminated timings

## Evidence

- [abstract] Wall-clock scaling with δt_nem: "the wall-clock time scales roughly as t sim ∝ 1/ δt nem , where δt nem represents the time synchronisation between the global and local scales"
- [readme] Test workflow: "Set-up initial conditions: `python tests/cluster_test/initialise_cluster.py` - To run Nemesis`` `python tests/cluster_test/run_cluster.py` with the flag `RUN_NEMESIS = 1`"
- [readme] Benchmark parameters: "End time: 0.1 Myr. This is a short enough time to allow any systematic errors to emerge while also remaining below the cluster's crossing time."
- [readme] Bridge timestep setting: "Bridge time: 500 yr."
- [readme] Controlled comparison: "Turn off galactic field + stellar evolution -- This will allow a better comparison in performance."
