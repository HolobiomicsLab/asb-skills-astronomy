---
name: particle-migration-jump-kick-handling
description: 'Use when reconstructing or extending a rung-based individual/adaptive
  N-body time-stepper: after rung re-assignment (acceleration criterion, eq. 42),
  a particle''s rung changes and its kick schedule must be bridged without desynchronising
  positions.'
when_to_use_negative:
- The simulation uses a single global time step (no rung hierarchy), so no migration
  between levels occurs
- The time-stepping limiter concerns fluid components rather than N-body particles
  (a buggy limiter for fluid components existed prior to the v0.2.0 fix)
- Modifying the long-range P³M/P^3M force computation rather than the short-range
  kick/rung schedule — the paper notes P³M is an inefficient option in CONCEPT
tools:
- name: CONCEPT 1.0
  role: Source code providing the rung-based hierarchical leapfrog time-stepper (arch_timestepping)
    whose jump-kick migration mechanism is reproduced
  repo: https://github.com/jmd-dk/concept/
- name: Cython
  role: Language used to implement the drift/kick scheduler and jump kicks, translating
    Python code to equivalent C
- name: NumPy
  role: Array operations for particle rung re-assignment and kick bookkeeping
  repo: https://github.com/numpy/numpy
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/mnras/stac568
    title: The cosmological simulation code CO<i>N</i>CEPT 1.0
  source_content_hash: sha256:54d77b7e307e0a0b1ee3659739597128772d2b9b4932719d518dbbfaf67ba2af
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/jmd-dk/concept
  tools:
  - CONCEPT 1.0
  - Cython
  - NumPy
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# particle-migration-jump-kick-handling

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Implements CONCEPT 1.0's mechanism for moving particles between adjacent rungs of the hierarchical leapfrog time-stepping scheme (levels ℓ with step size Δt/2^ℓ) using special 1/2-size and 3/4-size jump kicks, so that all particle positions remain mutually synchronised across rung changes.

## When to use

Use when reconstructing or extending a rung-based individual/adaptive N-body time-stepper: after rung re-assignment (acceleration criterion, eq. 42), a particle's rung changes and its kick schedule must be bridged without desynchronising positions.

## When NOT to use

- The simulation uses a single global time step (no rung hierarchy), so no migration between levels occurs
- The time-stepping limiter concerns fluid components rather than N-body particles (a buggy limiter for fluid components existed prior to the v0.2.0 fix)
- Modifying the long-range P³M/P^3M force computation rather than the short-range kick/rung schedule — the paper notes P³M is an inefficient option in CONCEPT

## Inputs

- CONCEPT 1.0 source tree (src, param, test directories) from github.com/jmd-dk/concept
- Parameter file defining a small test system (base time step Δt, softening lengths, η, number of rungs)
- Particle accelerations from the previous short-range kick (for eq. 42 rung assignment)

## Outputs

- Hierarchical leapfrog schedule with 1/2-size and 3/4-size jump kicks migrating particles between adjacent rungs
- Per-cycle rung assignment for every particle satisfying the eq. (42) acceleration criterion
- test_log file recording synchronisation and criterion checks after each base-step cycle

## How to apply

Within the hierarchical leapfrog integrator (arch_timestepping in CONCEPT's Cython/Python source), schedule drifts and kicks over n_rung rungs so that particles on rung ℓ receive kicks of size Δt/2^ℓ within a base step Δt. Re-evaluate each particle's rung after every cycle using the acceleration criterion of eq. (42): a particle must not accelerate across a fraction η of its softening length within its rung's time step, with the acceleration inferred from the previous short-range kick. When the criterion promotes or demotes a particle to an adjacent rung, apply the special 1/2-size and 3/4-size jump kicks that bridge the difference in kick schedules between the two rungs, keeping positions mutually synchronised at all times. Validate by running complete base-step cycles on a small test system configured via a parameter file, confirming after each cycle that all positions are synchronised and every rung assignment satisfies the acceleration criterion, recording checks in the test_log file.

## Related tools

- **CONCEPT 1.0** (Source code providing the rung-based hierarchical leapfrog time-stepper (arch_timestepping) whose jump-kick migration mechanism is reproduced) — https://github.com/jmd-dk/concept/
- **Cython** (Language used to implement the drift/kick scheduler and jump kicks, translating Python code to equivalent C)
- **NumPy** (Array operations for particle rung re-assignment and kick bookkeeping) — https://github.com/numpy/numpy

## Examples

```
"${concept}" -t "basic" 2>&1 | tee "${concept_test_log}"
```

## Evaluation signals

- After each complete base-step cycle, all particle positions are mutually synchronised regardless of rung history
- Every particle's rung assignment satisfies the eq. (42) acceleration criterion (no acceleration across fraction η of the softening length within the rung's Δt/2^ℓ step)
- Migrated particles only ever move between adjacent rungs (ℓ → ℓ±1), never skipping levels
- All checks are recorded in the test_log file when running the timestepper through complete base-step cycles
- Agreement of resulting power spectra with GADGET to well below the per mille level at large/intermediate scales and ~1% at k ~ 13 h/Mpc

## Limitations

- CONCEPT 1.0 uses the standard slab-decomposition of FFTW, which limits the number of CPU cores participating in the long-range force computation
- Load imbalance in strongly clustered systems degrades performance
- An erroneous time-stepping limiter for fluid components existed prior to the v0.2.0 fix
- CONCEPT is several times faster than GADGET-2 only for low to medium resolutions (k_Nyquist ≲ 5 Mpc/h); it is slower for higher-resolution simulations

## Evidence

- [methods] Rung assignment via acceleration criterion: [evidence span withheld — anchor status: not_found]
- [methods] Jump kicks keep positions synchronised: [evidence span withheld — anchor status: not_found]
- [methods] Post-cycle validation requirements: [evidence span withheld — anchor status: not_found]
