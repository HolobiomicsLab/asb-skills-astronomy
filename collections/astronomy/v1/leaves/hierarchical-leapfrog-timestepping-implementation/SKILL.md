---
name: hierarchical-leapfrog-timestepping-implementation
description: Use when implementing or re-implementing adaptive, individual-particle
  time integration in an N-body cosmology code, specifically the multi-rung leapfrog
  schedule and rung-assignment criterion of CONCEPT 1.0.
when_to_use_negative:
- The goal is long-range force computation or P³M/PM gravity, not timestepping — the
  paper notes P³M as inefficient and separate from the stepping scheme
- The target code uses a single global time step for all particles with no individual/adaptive
  stepping
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: CONCEPT 1.0
  role: Reference implementation whose hierarchical leapfrog timestepper (arch_timestepping)
    and rung-assignment criterion (eq. 42) are reconstructed
  repo: github.com/jmd-dk/concept
- name: Cython
  role: Translates the performance-critical timestepping code from Python to C
- name: Python
  role: Language in which the timestepper is implemented
  repo: github.com/numpy/numpy
- name: NumPy
  role: Numerical library used for array operations in the stepping and rung-assignment
    logic
  repo: github.com/numpy/numpy
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/mnras/stac568
    title: The cosmological simulation code CO<i>N</i>CEPT 1.0
  source_content_hash: sha256:914a941b44fdfb3b56810146335fc01d7091ed536ed93e57d4aa680cabc10309
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - CONCEPT 1.0
  - Cython
  - NumPy
  - Python
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# hierarchical-leapfrog-timestepping-implementation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Reconstruct CONCEPT 1.0's rung-based individual and adaptive particle time-stepping: a hierarchical leapfrog scheme in which particles on rung ℓ receive kicks of size Δt/2^ℓ and are assigned to rungs via an acceleration criterion (eq. 42) based on displacement across a fraction η of the softening length.

## When to use

Use when implementing or re-implementing adaptive, individual-particle time integration in an N-body cosmology code, specifically the multi-rung leapfrog schedule and rung-assignment criterion of CONCEPT 1.0.

## When NOT to use

- The goal is long-range force computation or P³M/PM gravity, not timestepping — the paper notes P³M as inefficient and separate from the stepping scheme
- The target code uses a single global time step for all particles with no individual/adaptive stepping

## Inputs

- CONCEPT source tree from github.com/jmd-dk/concept (src, param, test directories)
- Simulation parameter file configuring a small test system (n_rung, base step Δt, softening lengths)
- Particle accelerations from the previous short-range kick

## Outputs

- Hierarchical leapfrog timestepper (arch_timestepping) in Cython/Python scheduling drifts/kicks over rungs
- Rung assignment logic implementing eq. 42 with fraction-η-of-softening displacement threshold
- test_log recording per-cycle synchronisation and rung-criterion checks

## How to apply

Implement the hierarchical leapfrog integrator (arch_timestepping) in Cython/Python: schedule drifts and kicks over n_rung rungs so particles on rung ℓ receive kicks of size Δt/2^ℓ within a base step Δt. Assign each particle to a rung using the acceleration criterion of eq. 42 — a particle must not accelerate across a fraction η of its softening length within its rung's time step Δt/2^ℓ_i, with the acceleration inferred from the previous short-range kick — and re-evaluate rungs after every cycle. Implement the 1/2-size and 3/4-size jump kicks that migrate particles between adjacent rungs while keeping positions mutually synchronised. Validate by running a small parameter-file-configured system through complete base-step cycles, confirming after each cycle that all particle positions are mutually synchronised and that every rung assignment satisfies the acceleration criterion.

## Related tools

- **CONCEPT 1.0** (Reference implementation whose hierarchical leapfrog timestepper (arch_timestepping) and rung-assignment criterion (eq. 42) are reconstructed) — github.com/jmd-dk/concept
- **Cython** (Translates the performance-critical timestepping code from Python to C)
- **Python** (Language in which the timestepper is implemented) — github.com/numpy/numpy
- **NumPy** (Numerical library used for array operations in the stepping and rung-assignment logic) — github.com/numpy/numpy

## Examples

```
concept -t basic 2>&1 | tee concept_test_log
```

## Evaluation signals

- After every complete base-step cycle all particle positions are mutually synchronised
- Every particle's rung assignment satisfies the acceleration criterion of eq. 42 (no crossing of fraction η of the softening length within Δt/2^ℓ)
- Checks for both invariants are recorded in the test_log file after each cycle
- The timestepper completes full base-step cycles without error for the small test system defined in the parameter file
- Rung migrations only occur between adjacent rungs via the 1/2-size and 3/4-size jump kicks

## Limitations

- A buggy time-stepping limiter for fluid components existed prior to the v0.2.0 fix ('Erroneous time-stepping limiter for fluid components')
- Rung assignment depends on the acceleration inferred from the previous short-range kick, so accuracy of the criterion inherits errors from the prior kick
- Load imbalance in strongly clustered systems degrades performance when particles concentrate on deep rungs

## Evidence

- [other] Rung assignment is determined by an acceleration criterion: a particle must not accelerate across a fraction η of its softening length within its rung's time step Δt/2^ℓ_i, implemented as equation (42): [evidence span withheld — anchor status: not_found]
