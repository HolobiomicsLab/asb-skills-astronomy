---
name: runtime-benchmarking-and-profiling
description: Use when measuring how a dependency-aware incremental-update (caching)
  strategy reduces recomputation time versus recomputing all quantities, e.g. timing
  hmf's update() cascades.
when_to_use_negative:
- The goal is validating numerical accuracy of the HMF itself rather than the timing
  efficiency of the update mechanism
- The computation has no dependency graph or cached quantities, so incremental updates
  cannot save work and only full recomputation is possible
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: hmf
  role: Engine whose update() mechanism and dependency-tracked quantities (power,
    sigma, growth, dndlnm, ngtm) are benchmarked
  repo: https://github.com/halomod/hmf
- name: Python
  role: Language used to implement the timing harness and write cascading_update_benchmark.csv
- name: CAMB
  role: Computes the transfer function used to initialize the baseline MassFunction
    before benchmarking updates
  repo: https://github.com/joezuntz/pycamb
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2013.11.001
    title: 'HMFcalc: An online tool for calculating dark matter halo mass functions'
  source_content_hash: sha256:6340cc21c297a0f428aec92a46ac3c9dd108411a998741c585bf4483d50471aa
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/halomod/hmf
  tools:
  - CAMB
  - Python
  - hmf
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# runtime-benchmarking-and-profiling

## Summary

Benchmark the wall-clock benefit of hmf's cascading update() mechanism — which deletes only dependent quantities when a parameter changes — against full recomputation, reproducing the paper's reported ~50% and >94% processing-time reductions.

## When to use

Use when measuring how a dependency-aware incremental-update (caching) strategy reduces recomputation time versus recomputing all quantities, e.g. timing hmf's update() cascades.

## When NOT to use

- The goal is validating numerical accuracy of the HMF itself rather than the timing efficiency of the update mechanism
- The computation has no dependency graph or cached quantities, so incremental updates cannot save work and only full recomputation is possible

## Inputs

- hmf MassFunction instance with default parameters (CAMB transfer function, 4097-step k-grid, Romberg integration)
- Set of parameter updates to apply via update() (all parameters once for scenario a; redshift only for scenario b)
- Full-recomputation baseline timings for the same parameter changes

## Outputs

- cascading_update_benchmark.csv with per-scenario timings and computed reduction percentages
- Measured reduction percentages: ~50% (each parameter changed once) and >94% (redshift-only)

## How to apply

Load the hmf MassFunction engine with defaults (transfer function from CAMB, linear-spline log-space interpolation, re-gridding between k-bounds in 4097 steps, Romberg integration) and build the dependency graph over tracked quantities (power, sigma, growth, n_eff, fsigma, dndlnm, ngtm). In scenario (a), call update() changing each available parameter once and time the cascading update against full recomputation; in scenario (b), change only redshift so growth is rescaled rather than recomputed. The cascade works because update() deletes quantities that depend directly on the changed variable, and deletions propagate transitively so only affected quantities are recalculated. Record per-scenario timings and reduction percentages into cascading_update_benchmark.csv and compare against the reported values.

## Related tools

- **hmf** (Engine whose update() mechanism and dependency-tracked quantities (power, sigma, growth, dndlnm, ngtm) are benchmarked) — https://github.com/halomod/hmf
- **Python** (Language used to implement the timing harness and write cascading_update_benchmark.csv)
- **CAMB** (Computes the transfer function used to initialize the baseline MassFunction before benchmarking updates) — https://github.com/joezuntz/pycamb

## Examples

```
from hmf import MassFunction; mf = MassFunction(); import time; t=time.time(); mf.update(z=1.0); print(time.time()-t)
```

## Evaluation signals

- Scenario (a) reduction (each parameter changed once) reproduces the paper's reported ~50% processing-time saving
- Scenario (b) reduction (redshift-only change) exceeds 94%
- HMF values after cascading update() match those from full recomputation for the same parameter values
- cascading_update_benchmark.csv contains timings and reduction percentages for both scenarios

## Limitations

- The reported reductions are the paper's own test results, not an independent validation; absolute timings depend on machine and configuration
- Savings depend on which parameters are changed: changing a parameter that nearly all quantities depend on (e.g. the transfer-function cosmology) erodes the benefit, while redshift-only changes maximize it

## Evidence

- [methods] Cascading update via update() deletes dependents: [evidence span withheld — anchor status: not_found]
