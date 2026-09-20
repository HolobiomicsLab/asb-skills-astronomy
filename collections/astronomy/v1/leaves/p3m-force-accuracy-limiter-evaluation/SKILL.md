---
name: p3m-force-accuracy-limiter-evaluation
description: Use when reconstructing CONCEPT 1.0's global time-step scheme (e.g. Figure
  2) and you need the resolution-dependent Δln a branch driven by particle clustering,
  for box sizes like L_box ∈ {2, 1, 1/2} × ∛N Mpc/h.
when_to_use_negative:
- When only the resolution-independent limiters matter (fixed-Δa early/late branches
  or the Hubble limiter) — those need no particle dynamics and produce a single resolution-independent
  curve
- When studying fluid/linear-species time stepping in CONCEPT rather than the N-body
  particle P3M limiter (a separate, historically buggy limiter for fluid components
  was fixed in v0.2.0)
- When using GADGET-2/GADGET-4 instead of CONCEPT — the P3M limiter scheme and its
  Figure 2 behaviour are specific to CONCEPT 1.0's time-step machinery
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3450
tools:
- name: CONCEPT 1.0
  role: Simulation code whose P3M time-stepping limiter is evaluated; parameter files
    configured per box size
  repo: https://github.com/jmd-dk/concept
- name: NumPy
  role: Pointwise combination of limiter curves via elementwise minimum to obtain
    the global Δln a(a)
  repo: https://github.com/numpy/numpy
- name: Matplotlib
  role: Plotting the three Δln a(a) curves against scale factor (Figure 2 reproduction)
- name: class
  role: Linear Einstein–Boltzmann code CONCEPT's background/cosmology output is designed
    to agree with, used to set the Table 1 ΛCDM parameters
  repo: https://github.com/lesgourg/class_public
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/mnras/stac568
    title: The cosmological simulation code CO<i>N</i>CEPT 1.0
  source_content_hash: sha256:9a45b1fc247f8be14365f0ab92d1a7d4cdfa4d9631b4a06ccf684ef63835e7d9
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/jmd-dk/concept
  tools:
  - CONCEPT 1.0
  - Matplotlib
  - NumPy
  - Python
  - class
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3450
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# p3m-force-accuracy-limiter-evaluation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Evaluates the non-linear P3M time-stepping limiter of CONCEPT 1.0, which caps the global time step Δln a whenever near-field P3M force corrections lose accuracy in clustered particle configurations. It is the only dynamical limiter — and hence the only one whose allowed step depends on simulation resolution.

## When to use

Use when reconstructing CONCEPT 1.0's global time-step scheme (e.g. Figure 2) and you need the resolution-dependent Δln a branch driven by particle clustering, for box sizes like L_box ∈ {2, 1, 1/2} × ∛N Mpc/h.

## When NOT to use

- When only the resolution-independent limiters matter (fixed-Δa early/late branches or the Hubble limiter) — those need no particle dynamics and produce a single resolution-independent curve
- When studying fluid/linear-species time stepping in CONCEPT rather than the N-body particle P3M limiter (a separate, historically buggy limiter for fluid components was fixed in v0.2.0)
- When using GADGET-2/GADGET-4 instead of CONCEPT — the P3M limiter scheme and its Figure 2 behaviour are specific to CONCEPT 1.0's time-step machinery

## Inputs

- CONCEPT simulation parameter files from github.com/Aarhus Cosmology/concept1.0-data (Table 1 ΛCDM cosmology plus particle load)
- Box-size configurations L_box ∈ {2, 1, 1/2} × ∛N Mpc/h
- Particle displacement / clustering state as a function of scale factor a
- Companion limiter branches to take the pointwise minimum against: fixed-Δa branches (early/late, transitioning around matter–Λ equality) and the Hubble limiter

## Outputs

- P3M-limiter curve Δln a_P3M(a) per box size
- Global Δln a(a) curve per box after pointwise minimum with all other limiters
- Figure 2-style Matplotlib plot of Δln a vs scale factor with one curve per box size

## How to apply

Load the ΛCDM particle-load parameter files for each box size from the public concept1.0-data repository and configure the Table 1 cosmology. Evaluate the P3M limiter as a function of scale factor: it shrinks the allowed step when near-field force corrections lose accuracy in strongly clustered configurations, so compute the maximum particle displacement relative to the mean inter-particle separation per step. Unlike the fixed-Δa branches and the Hubble limiter (which caps Δln a per Hubble time), this limiter must be recomputed from the actual particle dynamics at each step. Combine it pointwise with the other limiters by taking the minimum allowed Δln a across all limiters to obtain the global Δln a(a) curve per box. Validate that the P3M-limited step strictly decreases with resolution — the ordering across L_box ∈ {2, 1, 1/2} × ∛N Mpc/h must be monotonic (2 > 1 > 1/2), with smaller boxes (higher resolution) permitting lower time steps.

## Related tools

- **CONCEPT 1.0** (Simulation code whose P3M time-stepping limiter is evaluated; parameter files configured per box size) — https://github.com/jmd-dk/concept
- **NumPy** (Pointwise combination of limiter curves via elementwise minimum to obtain the global Δln a(a)) — https://github.com/numpy/numpy
- **Matplotlib** (Plotting the three Δln a(a) curves against scale factor (Figure 2 reproduction))
- **class** (Linear Einstein–Boltzmann code CONCEPT's background/cosmology output is designed to agree with, used to set the Table 1 ΛCDM parameters) — https://github.com/lesgourg/class_public

## Examples

```
import numpy as np; dlna_global = np.minimum.reduce([dlna_dyn, dlna_fixed_early, dlna_fixed_late, dlna_hubble, dlna_p3m])
```

## Evaluation signals

- P3M-limited Δln a shows strict monotonic ordering with box size: 2 × ∛N Mpc/h > 1 × ∛N Mpc/h > 1/2 × ∛N Mpc/h (smaller boxes = higher resolution = smaller allowed steps)
- The P3M limiter is the only branch that changes between the three box-size curves; all other limiter branches overlap across resolutions
- In the combined global Δln a(a) curve, the fixed-Δa branches exhibit the matter–Λ equality transition feature
- The P3M-limited step shrinks as clustering grows toward late times, since it is triggered by loss of accuracy of near-field force corrections in clustered configurations

## Limitations

- The P3M limiter requires actual particle dynamics, so it cannot be evaluated analytically without running (or replaying) the simulation at each box size/resolution
- P3M gravity in CONCEPT 1.0 is noted as inefficient relative to alternatives, which matters if re-running simulations to trace the limiter
- A buggy time-stepping limiter for fluid components existed prior to the v0.2.0 fix — use CONCEPT ≥ 1.0 (post-fix) for limiter studies

## Evidence

- [other] P3M limiter is the only dynamical, resolution-dependent branch: [evidence span withheld — anchor status: not_found]
- [other] Fixed-Δa limiter structure and combination rule: [evidence span withheld — anchor status: not_found]
- [other] P3M limiter trigger condition: [evidence span withheld — anchor status: not_found]
