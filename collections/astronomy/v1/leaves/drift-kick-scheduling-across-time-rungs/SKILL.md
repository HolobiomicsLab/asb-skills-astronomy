---
name: drift-kick-scheduling-across-time-rungs
description: Use when implementing or reproducing an individual/adaptive particle
  time-stepping scheme from CONCEPT 1.0, e.g. scheduling drifts and kicks over multiple
  rungs and assigning particles to rungs via the eq. 42 acceleration criterion.
when_to_use_negative:
- You only need a uniform global time step for all particles (no adaptive rung structure
  required)
- You need long-range force computation at very high core counts — the FFTW slab-decomposition
  limits the number of CPU cores participating, a performance limitation separate
  from the timestepping scheme
tools:
- name: CONCEPT 1.0
  role: Source code providing the hierarchical leapfrog timestepper (arch_timestepping),
    eq. 42 rung-assignment criterion, and test/param directories to reconstruct
  repo: github.com/jmd-dk/concept
- name: Cython
  role: Implementation language in which the drift/kick scheduling over rungs is written
    for performance
- name: Python
  role: Host language of CONCEPT, used for the top-level timestepper driver and parameter
    configuration
- name: NumPy
  role: Numerical library used alongside Cython for particle state manipulation in
    the timestepper
  repo: github.com/numpy/numpy
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/mnras/stac568
    title: The cosmological simulation code CO<i>N</i>CEPT 1.0
  source_content_hash: sha256:0ecdc7ccaf0d59680be7b0e73898cf168abcef227fa396599fa2e1e35e91eacb
schema_version: 0.2.0
metadata:
  license_tier: restricted
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
yes_rate: 0.8333333333333334
n_claims: 6
---

# drift-kick-scheduling-across-time-rungs

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Reconstructs CONCEPT 1.0's hierarchical leapfrog time-stepping, in which individual particles sit on rungs ℓ receiving kicks of size Δt/2^ℓ, with rung assignment driven by the acceleration criterion of eq. 42. Needed to reproduce adaptive per-particle time resolution in cosmological N-body simulations.

## When to use

Use when implementing or reproducing an individual/adaptive particle time-stepping scheme from CONCEPT 1.0, e.g. scheduling drifts and kicks over multiple rungs and assigning particles to rungs via the eq. 42 acceleration criterion.

## When NOT to use

- You only need a uniform global time step for all particles (no adaptive rung structure required)
- You need long-range force computation at very high core counts — the FFTW slab-decomposition limits the number of CPU cores participating, a performance limitation separate from the timestepping scheme

## Inputs

- CONCEPT 1.0 source tree (src, param, test directories) from github.com/jmd-dk/concept
- Simulation parameter file defining a small test system
- Particle accelerations from the previous short-range kick (used for eq. 42 rung assignment)

## Outputs

- Hierarchical leapfrog timestepper (arch_timestepping) implemented in Cython/Python with scheduled drifts/kicks over n_rung rungs
- test_log file recording, after each base-step cycle, that all particle positions are mutually synchronised and every rung assignment satisfies the acceleration criterion

## How to apply

Obtain the CONCEPT 1.0 source tree from github.com/jmd-dk/concept (src, param, test directories) and implement the hierarchical leapfrog integrator (arch_timestepping) in Cython/Python, scheduling drifts and kicks over n_rung rungs so that particles on rung ℓ receive kicks of size Δt/2^ℓ. Assign rungs with the eq. 42 criterion: a particle must not accelerate across a fraction η of its softening length within its rung's time step, using the acceleration inferred from the previous short-range kick, re-evaluated after every cycle. Migrate particles between adjacent rungs only, using the 1/2-size and 3/4-size jump kicks that keep all particle positions mutually synchronised. Configure a small test system through a parameter file and drive the timestepper through complete base-step cycles, logging validation checks to the test_log file.

## Related tools

- **CONCEPT 1.0** (Source code providing the hierarchical leapfrog timestepper (arch_timestepping), eq. 42 rung-assignment criterion, and test/param directories to reconstruct) — github.com/jmd-dk/concept
- **Cython** (Implementation language in which the drift/kick scheduling over rungs is written for performance)
- **Python** (Host language of CONCEPT, used for the top-level timestepper driver and parameter configuration)
- **NumPy** (Numerical library used alongside Cython for particle state manipulation in the timestepper) — github.com/numpy/numpy

## Examples

```
"${concept}" -t "basic" 2>&1 | tee "${concept_test_log}"
```

## Evaluation signals

- After every complete base-step cycle, all particle positions are mutually synchronised
- Every particle's rung assignment satisfies the eq. 42 acceleration criterion (no acceleration across a fraction η of the softening length within its rung's step Δt/2^ℓ)
- Particles migrate only between adjacent rungs, via the 1/2-size and 3/4-size jump kicks
- All validation checks are recorded in the test_log file, matching the code's own test-suite practice (e.g. 'concept -t basic' output teed to a test log)
- Rung re-evaluation occurs after every cycle, with particles on rung ℓ receiving kicks of exactly Δt/2^ℓ

## Limitations

- CONCEPT's P³M gravity is noted as inefficient, so the short-range accelerations feeding the eq. 42 criterion carry that performance caveat
- A buggy time-stepping limiter for fluid components existed prior to the v0.2.0 fix, so fluid (non-particle) components in older versions may not have been stepped correctly

## Evidence

- [full_text] Rung assignment via acceleration criterion: [evidence span withheld — anchor status: not_found]
- [abstract] Scaling context for large simulations: [evidence span withheld — anchor status: not_found]
