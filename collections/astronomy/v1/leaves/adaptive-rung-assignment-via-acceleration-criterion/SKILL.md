---
name: adaptive-rung-assignment-via-acceleration-criterion
description: Use when implementing or validating adaptive, individual-particle time-stepping
  in an N-body cosmology code, or when verifying that per-particle rung assignments
  keep positions mutually synchronised over a base-step cycle.
when_to_use_negative:
- The goal is a single global (non-individual) time step — the rung scheme only pays
  off for adaptive per-particle stepping.
- The target is fluid/relativistic component time-stepping rather than particle rungs;
  the paper notes an 'Erroneous time-stepping limiter for fluid components' prior
  to the v0.2.0 fix.
- You are modelling strongly clustered systems at very high resolution, where CONCEPT
  1.0 is slower than GADGET and load imbalance degrades performance.
tools:
- name: CONCEPT 1.0
  role: Reference implementation of the rung-based hierarchical leapfrog timestepper
    whose scheme is reconstructed
  repo: github.com/jmd-dk/concept
- name: Cython
  role: Language used to implement the hierarchical leapfrog integrator (arch_timestepping)
  repo: github.com/jmd-dk/concept
- name: Python
  role: Host language of the CONCEPT code and its parameter-file-driven timestepper
  repo: github.com/jmd-dk/concept
- name: NumPy
  role: Numerical library used for array operations in the Python/Cython implementation
  repo: github.com/numpy/numpy
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/mnras/stac568
    title: The cosmological simulation code CO<i>N</i>CEPT 1.0
  source_content_hash: sha256:4da3860d5378f022de38da4ed9cd21671eb8e0bb2ff8d387e70823efabd605ec
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
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# adaptive-rung-assignment-via-acceleration-criterion

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Reconstructs CONCEPT 1.0's hierarchical leapfrog time-stepping scheme in which individual particles are assigned to rungs ℓ (step size Δt/2^ℓ) using the acceleration criterion of eq. 42: a particle must not accelerate across a fraction η of its softening length within its rung's time step.

## When to use

Use when implementing or validating adaptive, individual-particle time-stepping in an N-body cosmology code, or when verifying that per-particle rung assignments keep positions mutually synchronised over a base-step cycle.

## When NOT to use

- The goal is a single global (non-individual) time step — the rung scheme only pays off for adaptive per-particle stepping.
- The target is fluid/relativistic component time-stepping rather than particle rungs; the paper notes an 'Erroneous time-stepping limiter for fluid components' prior to the v0.2.0 fix.
- You are modelling strongly clustered systems at very high resolution, where CONCEPT 1.0 is slower than GADGET and load imbalance degrades performance.

## Inputs

- CONCEPT source tree from github.com/jmd-dk/concept (src, param, test directories)
- Small test-system parameter file (particle softening lengths, η, n_rung, base step Δt)
- Per-particle accelerations from the previous short-range kick

## Outputs

- Per-particle rung assignments ℓ_i satisfying eq. 42
- Drift/kick schedule of jump kicks (1/2-size and 3/4-size) over the base-step cycle
- test_log entries recording synchronisation and criterion checks per cycle

## How to apply

Implement the hierarchical leapfrog integrator (arch_timestepping) in Cython/Python: schedule drifts and kicks over n_rung rungs, with particles on rung ℓ receiving kicks of size Δt/2^ℓ. Assign rungs with the acceleration criterion of eq. 42 — the acceleration inferred from the particle's previous short-range kick must not carry it across a fraction η of its softening length within Δt/2^ℓ_i — and re-evaluate every particle's rung after each cycle. Use the 1/2-size and 3/4-size jump kicks to migrate particles between adjacent rungs while keeping all positions mutually synchronised. Configure a small test system via a CONCEPT parameter file and run the timestepper through complete base-step cycles, confirming after each cycle that positions are synchronised and that each rung assignment satisfies the criterion, recording the checks in the test_log file.

## Related tools

- **CONCEPT 1.0** (Reference implementation of the rung-based hierarchical leapfrog timestepper whose scheme is reconstructed) — github.com/jmd-dk/concept
- **Cython** (Language used to implement the hierarchical leapfrog integrator (arch_timestepping)) — github.com/jmd-dk/concept
- **Python** (Host language of the CONCEPT code and its parameter-file-driven timestepper) — github.com/jmd-dk/concept
- **NumPy** (Numerical library used for array operations in the Python/Cython implementation) — github.com/numpy/numpy

## Examples

```
"${concept}" -t "basic" 2>&1 | tee "${concept_test_log}"
```

## Evaluation signals

- After every base-step cycle, all particle positions are mutually synchronised.
- Every particle's rung assignment satisfies the eq. 42 acceleration criterion (no crossing of fraction η of the softening length within Δt/2^ℓ_i).
- Checks after each cycle are recorded in the test_log file.
- The timestepper completes complete base-step cycles on a small parameter-file-configured test system without errors.
- Downstream correctness: CONCEPT and GADGET power spectra agree well below the per mille level at large and intermediate scales.

## Limitations

- CONCEPT 1.0 is several times faster than GADGET-2 only for low to medium resolutions (k_Nyquist < 5 Mpc/h); it is slower for higher-resolution simulations.
- Performance is limited by load imbalance in strongly clustered systems and the FFTW slab-decomposition, which caps core counts in the long-range force computation.
- A buggy (erroneous) time-stepping limiter for fluid components existed prior to the v0.2.0 fix, so older versions must not be relied on for fluid time-stepping.

## Evidence

- [full_text] Rung assignment is determined by an acceleration criterion: a particle must not accelerate across a fraction η of its softening length within its rung's time step Δt/2^ℓ_i, implemented as equation (42): [evidence span withheld — anchor status: not_found]
- [abstract] concept makes heavy use of Cython (Behnel et al. 2011), which translates Python code to equivalent C code: "concept makes heavy use of Cython (Behnel et al. 2011), which translates Python code to equivalent C code"
- [abstract] At large and intermediate scales the codes agree to well below the per mille level: "At large and intermediate scales the codes agree to well below the per mille level"
- [discussion] Erroneous time-stepping limiter for fluid components.: "Erroneous time-stepping limiter for fluid components."
