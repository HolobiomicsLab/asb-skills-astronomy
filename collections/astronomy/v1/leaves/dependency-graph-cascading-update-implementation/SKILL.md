---
name: dependency-graph-cascading-update-implementation
description: Use when a parameterized scientific calculation must be re-run after
  changing one or more inputs and only the dependent quantities should be recomputed
  rather than the entire pipeline.
when_to_use_negative:
- The user's changed parameters invalidate the underlying transfer function itself
  (e.g. cosmology changes requiring new CAMB runs), so savings shrink toward full
  recomputation
- A one-off calculation where no parameters will be updated after initial computation
  — the dependency-tracking machinery adds overhead without benefit
- Quantities outside the tracked dependency graph (untracked cached state) need to
  be updated, since update() only deletes quantities it knows depend on the changed
  variables
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: hmf
  role: Engine providing the update() method with dependency tracking over tracked
    quantities (dndlnm, ngtm, sigma, growth, power, etc.)
  repo: https://github.com/steven-murray/HMF
- name: Python
  role: Implementation language used to build the dependency graph, run update() benchmarks,
    and record timings
- name: CAMB
  role: Computes the transfer function that anchors the dependency graph; avoided
    on redshift-only updates via growth rescaling
  repo: https://github.com/joezuntz/pycamb
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2013.11.001
    title: 'HMFcalc: An online tool for calculating dark matter halo mass functions'
  source_content_hash: sha256:5635bedbe39adf2e012a8646db8b98cf8de998a5210d829f65b06e42e068f148
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/steven-murray/HMF
  tools:
  - CAMB
  - Python
  - hmf
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# dependency-graph-cascading-update-implementation

## Summary

Implements hmf's cascading parameter-update mechanism, in which update() deletes only tracked quantities (power, sigma, growth, n_eff, fsigma, dndlnm, ngtm) that depend directly or transitively on a changed input, avoiding full recomputation. Benchmarks report ~50% processing-time reduction when every parameter is changed once and >94% when only redshift changes.

## When to use

Use when a parameterized scientific calculation must be re-run after changing one or more inputs and only the dependent quantities should be recomputed rather than the entire pipeline.

## When NOT to use

- The user's changed parameters invalidate the underlying transfer function itself (e.g. cosmology changes requiring new CAMB runs), so savings shrink toward full recomputation
- A one-off calculation where no parameters will be updated after initial computation — the dependency-tracking machinery adds overhead without benefit
- Quantities outside the tracked dependency graph (untracked cached state) need to be updated, since update() only deletes quantities it knows depend on the changed variables

## Inputs

- hmf MassFunction/Perturbations engine instance initialized with default parameters (CAMB transfer function, log-space linear-spline interpolation, 4097-step k-grid, Romberg integration)
- Dictionary of parameter changes passed as keyword arguments to update()
- Benchmark scenarios: (a) each available parameter changed once; (b) redshift-only change

## Outputs

- Recomputed dependent quantities (power, sigma, growth, n_eff, fsigma, dndlnm, ngtm) consistent with the new parameters
- Per-scenario processing times for cascading update vs full recomputation
- Reduction percentages, e.g. written to cascading_update_benchmark.csv

## How to apply

Build a dependency graph over the tracked quantities (power, sigma, growth, n_eff, fsigma, dndlnm, ngtm) so that quantities depending directly on a changed variable are deleted, and deletions cascade transitively to their dependents. Expose the mechanism through an update() method accepting relevant keyword arguments, so e.g. changing redshift only deletes growth-dependent quantities and rescales them rather than recomputing the CAMB transfer function. Benchmark scenario (a): update each available parameter one time, timing the cascading update against full recomputation; scenario (b): change only redshift, timing the growth-rescaling update against full recomputation. The hmf engine loads via CAMB with the transfer function interpolated by linear splines in log-space, re-gridded between k-bounds in 4097 steps, and uses Romberg integration for the cumulative HMF. Record per-scenario timings and reduction percentages (e.g. to cascading_update_benchmark.csv); validation reproduces the paper's reported ~50% reduction (all parameters changed once) and >94% reduction (redshift-only change).

## Related tools

- **hmf** (Engine providing the update() method with dependency tracking over tracked quantities (dndlnm, ngtm, sigma, growth, power, etc.)) — https://github.com/steven-murray/HMF
- **Python** (Implementation language used to build the dependency graph, run update() benchmarks, and record timings)
- **CAMB** (Computes the transfer function that anchors the dependency graph; avoided on redshift-only updates via growth rescaling) — https://github.com/joezuntz/pycamb

## Examples

```
mf = hmf.MassFunction(); t0=time(); mf.update(z=1.0); print(time()-t0)  # vs full recompute: hmf.MassFunction(z=1.0)
```

## Evaluation signals

- Measured processing-time reduction is approximately 50% when each available parameter is changed once, matching the paper's reported value
- Measured reduction exceeds 94% for the redshift-only scenario, where only the growth factor rescaling is recomputed
- Quantities recomputed after update() are bitwise/numerically identical to those from a full recomputation from scratch
- The deletion set after an update matches the transitive closure of the dependency graph from the changed variables (e.g. changing z deletes growth-dependent quantities but not the transfer function)
- Benchmark CSV contains per-scenario timings and reduction percentages for both scenarios (a) and (b)

## Limitations

- Reported speedups are the paper's own benchmark results, not an independent validation; absolute timings depend on the machine and default parameters (CAMB transfer, 4097-step k-grid, Romberg integration)
- Savings depend on which parameters are changed: redshift-only updates yield >94% reduction, but changing parameters that invalidate the transfer function recovers little of the saving
- The mechanism relies on complete dependency tracking; any quantity outside the tracked graph will not be invalidated, risking stale results

## Evidence

- [methods] update deletes only dependents: [evidence span withheld — anchor status: not_found]
- [readme] README caching claim: [evidence span withheld — anchor status: external_doc]
