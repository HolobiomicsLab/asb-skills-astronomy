---
name: matter-lambda-equality-diagnostics
description: 'Use when reproducing or validating CONCEPT 1.0 Figure 2 time-step curves:
  you must identify which limiter (fixed-Δa, Hubble, dynamical, or non-linear P3M)
  governs Δln a at each scale factor and check the equality-transition feature.'
when_to_use_negative:
- The goal is a power-spectrum code comparison (e.g. CONCEPT vs GADGET-2/GADGET-4
  agreement at k ~ 13 h/Mpc) rather than time-step limiter behaviour.
- The analysis targets performance scaling (weak/strong scaling up to 1024 CPU cores)
  or code installation/build diagnostics rather than the time-stepping scheme.
- Fluid-component time-stepping is the target in releases prior to v0.2.0, which contained
  an erroneous time-stepping limiter for fluid components.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: CONCEPT 1.0
  role: Simulation code whose parameter files and limiter definitions define the Δln
    a(a) curves being diagnosed
  repo: https://github.com/jmd-dk/concept
- name: NumPy
  role: Combines the individual limiter curves pointwise by taking the minimum allowed
    Δln a
  repo: https://github.com/numpy/numpy
- name: Matplotlib
  role: Plots the three Δln a(a) curves against scale factor for the three box sizes
- name: class
  role: Linear Einstein-Boltzmann solver whose background (e.g. matter–Λ equality)
    the limiter scheme is designed to agree with
  repo: https://github.com/lesgourg/class_public
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/mnras/stac568
    title: The cosmological simulation code CO<i>N</i>CEPT 1.0
  source_content_hash: sha256:2f7b543edcba368ba9b23c1e3b337249214d7097da55665e11f975b55600326d
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# matter-lambda-equality-diagnostics

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Diagnoses the matter–Λ equality transition in the CONCEPT 1.0 global time-step scheme by evaluating the fixed-Δa limiter branches (constant early and late branches transitioning around matter–Λ equality), combining all limiters pointwise with NumPy, and verifying the transition feature and resolution ordering in Figure 2-style Δln a(a) curves.

## When to use

Use when reproducing or validating CONCEPT 1.0 Figure 2 time-step curves: you must identify which limiter (fixed-Δa, Hubble, dynamical, or non-linear P3M) governs Δln a at each scale factor and check the equality-transition feature.

## When NOT to use

- The goal is a power-spectrum code comparison (e.g. CONCEPT vs GADGET-2/GADGET-4 agreement at k ~ 13 h/Mpc) rather than time-step limiter behaviour.
- The analysis targets performance scaling (weak/strong scaling up to 1024 CPU cores) or code installation/build diagnostics rather than the time-stepping scheme.
- Fluid-component time-stepping is the target in releases prior to v0.2.0, which contained an erroneous time-stepping limiter for fluid components.

## Inputs

- CONCEPT 1.0 simulation parameter files (from github.com/Aarhus Cosmology/concept1.0-data) configured with the Table 1 ΛCDM cosmology
- Particle loads for box sizes L_box ∈ {2, 1, 1/2} × ∛N Mpc/h at each simulation resolution
- Per-limiter Δln a(a) arrays: fixed-Δa (early and late branches), Hubble, dynamical, and non-linear P3M limiters

## Outputs

- Pointwise-minimum global time-step curve Δln a(a) per box size (NumPy arrays)
- Matplotlib figure with three Δln a(a) curves showing the matter–Λ equality transition and P3M-limiter resolution ordering
- Determination of which limiter governs the global time step at each scale factor

## How to apply

Load the CONCEPT simulation parameter files from the public concept1.0-data repository and configure the Table 1 ΛCDM cosmology and particle load for each box size L_box ∈ {2, 1, 1/2} × ∛N Mpc/h. Evaluate the two constant fixed-Δa branches as functions of scale factor, with the transition between them placed at matter–Λ equality, and separately evaluate the Hubble limiter (caps Δln a per Hubble time), the dynamical limiter (largest particle displacement relative to the mean inter-particle separation), and the non-linear P3M limiter per box size. Combine all limiters pointwise by taking the minimum allowed Δln a with NumPy to obtain Δln a(a) per box, then plot the three box-size curves against scale factor with Matplotlib. Judge correctness by the presence of the matter–Λ equality transition between the fixed-Δa branches and by the strict monotonic ordering of P3M-limited steps with box size (2 > 1 > 1/2 ∛N Mpc/h), since smaller boxes mean higher resolution and the P3M limiter is the only dynamics-dependent limiter.

## Related tools

- **CONCEPT 1.0** (Simulation code whose parameter files and limiter definitions define the Δln a(a) curves being diagnosed) — https://github.com/jmd-dk/concept
- **NumPy** (Combines the individual limiter curves pointwise by taking the minimum allowed Δln a) — https://github.com/numpy/numpy
- **Matplotlib** (Plots the three Δln a(a) curves against scale factor for the three box sizes)
- **class** (Linear Einstein-Boltzmann solver whose background (e.g. matter–Λ equality) the limiter scheme is designed to agree with) — https://github.com/lesgourg/class_public

## Examples

```
dlna = np.minimum(np.minimum(dlna_fixed_early, dlna_fixed_late), np.minimum(dlna_hubble, dlna_p3m)); plt.plot(a, dlna)
```

## Evaluation signals

- The produced figure displays the matter–Λ equality transition between the constant early-time and late-time fixed-Δa branches.
- P3M-limited time steps show strict monotonic ordering with box size: 2 > 1 > 1/2 × ∛N Mpc/h (smaller boxes, higher resolution, lower allowed steps).
- The combined curve equals the pointwise minimum across all limiters at every scale factor (no limiter branch exceeds the minimum envelope).
- Only the non-linear P3M limiter curve varies with particle dynamics/resolution; the fixed-Δa and Hubble limiter branches are resolution-independent.

## Limitations

- Only the non-linear P3M limiter depends on particle dynamics and hence simulation resolution; the fixed-Δa and Hubble limiters are resolution-independent and cannot diagnose resolution effects.
- An erroneous time-stepping limiter for fluid components existed prior to the v0.2.0 release, so diagnostics on fluid time-stepping are unreliable for older versions.
- Results depend on the Table 1 ΛCDM cosmology configuration; the equality-transition location shifts with different background cosmologies.

## Evidence

- [abstract] simulation parameter files for recreating the simulations producing this data — are made publicly available at github.com/Aarhus Cosmology/concept1.0-data: [evidence span withheld — anchor status: not_found]
- [intro] CO*N*CEPT (COsmological N-body CodE in PyThon) is a free and open-source simulation code for cosmological structure formation.: [evidence span withheld — anchor status: not_found]
