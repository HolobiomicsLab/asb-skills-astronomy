---
name: cosmological-time-step-limiter-analysis
description: Use when analysing or reproducing how the allowed global time step Δln
  a in a cosmological N-body code (specifically CONCEPT 1.0) evolves across cosmic
  history and which limiter governs at each scale factor and resolution.
when_to_use_negative:
- Input is an already-run CONCEPT simulation output (snapshots/power spectra) rather
  than parameter files and limiter definitions — this skill analyses the time-step
  scheme, not simulation results.
- The analysis targets GADGET-2/GADGET-4 time-stepping; only the CONCEPT 1.0 limiter
  definitions (dynamical, fixed-Δa, Hubble, non-linear P3M) are covered here.
- Fluid-component time-stepping in CONCEPT versions prior to v0.2.0, which used an
  erroneous time-stepping limiter for fluid components (fixed in v0.2.0).
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: CONCEPT 1.0
  role: Source of the time-step limiter scheme and simulation parameter files being
    reconstructed and reproduced
  repo: https://github.com/jmd-dk/concept
- name: Python
  role: Implementation language for evaluating and combining the limiter functions
- name: NumPy
  role: Pointwise combination of the limiters (minimum allowed Δln a) as arrays over
    scale factor
  repo: https://github.com/numpy/numpy
- name: Matplotlib
  role: Plotting the three Δln a(a) curves against scale factor
  repo: https://github.com/matplotlib/matplotlib
- name: class
  role: Linear Einstein-Boltzmann code providing background quantities (e.g. Hubble
    time) consistent with the simulation
  repo: https://github.com/lesgourg/class_public
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/mnras/stac568
    title: The cosmological simulation code CO<i>N</i>CEPT 1.0
  source_content_hash: sha256:7579d18f26f795e3242eaa742ebbf904409b39b6975304939b0faede2e7d04ad
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
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# cosmological-time-step-limiter-analysis

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Reconstructs the global time-step limiter scheme of CONCEPT 1.0 — the dynamical, fixed-Δa, Hubble, and non-linear P3M limiters — and evaluates Δln a(a) for the Table 1 ΛCDM cosmology at three box sizes, reproducing the Figure 2 curves.

## When to use

Use when analysing or reproducing how the allowed global time step Δln a in a cosmological N-body code (specifically CONCEPT 1.0) evolves across cosmic history and which limiter governs at each scale factor and resolution.

## When NOT to use

- Input is an already-run CONCEPT simulation output (snapshots/power spectra) rather than parameter files and limiter definitions — this skill analyses the time-step scheme, not simulation results.
- The analysis targets GADGET-2/GADGET-4 time-stepping; only the CONCEPT 1.0 limiter definitions (dynamical, fixed-Δa, Hubble, non-linear P3M) are covered here.
- Fluid-component time-stepping in CONCEPT versions prior to v0.2.0, which used an erroneous time-stepping limiter for fluid components (fixed in v0.2.0).

## Inputs

- CONCEPT 1.0 simulation parameter files from the concept1.0-data repository (github.com/Aarhus Cosmology/concept1.0-data)
- Table 1 ΛCDM cosmological parameters
- Particle load specifications for L_box ∈ {2, 1, 1/2} × 3√N Mpc/h
- Linear background quantities from class (Blas, Lesgourgues & Tram 2011) for the Hubble limiter

## Outputs

- Δln a(a) curves — combined global time step as a function of scale factor, one per box size (Figure 2 reproduction)
- Per-limiter Δln a contributions (dynamical, fixed-Δa early/late branches, Hubble, non-linear P3M)
- Matplotlib figure showing the three box-size curves and the matter–Λ equality transition

## How to apply

Load the simulation parameter files from the public concept1.0-data repository and configure the Table 1 ΛCDM cosmology with particle loads for L_box ∈ {2, 1, 1/2} × 3√N Mpc/h. Evaluate each limiter as a function of scale factor: the dynamical limiter restricts each step by the largest particle displacement relative to the mean inter-particle separation; the fixed-Δa limiters consist of a constant early-time branch and a constant late-time branch transitioning around matter–Λ equality; the Hubble limiter caps Δln a per Hubble time; and the non-linear P3M limiter shrinks the step when near-field force corrections lose accuracy in clustered configurations. Combine the limiters pointwise by taking the minimum allowed Δln a (with NumPy) and plot the resulting Δln a(a) curves per box size with Matplotlib. Validate that the fixed-Δa branches show the matter–Λ equality transition and that the P3M-limited steps are strictly ordered with box size (2 > 1 > 1/2 × 3√N Mpc/h), since only the non-linear P3M limiter depends on particle dynamics and hence simulation resolution.

## Related tools

- **CONCEPT 1.0** (Source of the time-step limiter scheme and simulation parameter files being reconstructed and reproduced) — https://github.com/jmd-dk/concept
- **Python** (Implementation language for evaluating and combining the limiter functions)
- **NumPy** (Pointwise combination of the limiters (minimum allowed Δln a) as arrays over scale factor) — https://github.com/numpy/numpy
- **Matplotlib** (Plotting the three Δln a(a) curves against scale factor) — https://github.com/matplotlib/matplotlib
- **class** (Linear Einstein-Boltzmann code providing background quantities (e.g. Hubble time) consistent with the simulation) — https://github.com/lesgourg/class_public

## Examples

```
import numpy as np; dlna = np.minimum.reduce([dyn(a), fixed_early(a), fixed_late(a), hubble(a), p3m(a, L_box)]); import matplotlib.pyplot as plt; plt.loglog(a, dlna)
```

## Evaluation signals

- The produced figure displays the matter–Λ equality transition between the two constant fixed-Δa branches.
- The P3M-limited steps are monotonically ordered with box size: L_box = 2 × 3√N Mpc/h > 1 × 3√N Mpc/h > 1/2 × 3√N Mpc/h (smaller boxes permit lower allowed time steps).
- The combined Δln a(a) equals the pointwise minimum of all evaluated limiters at every scale factor.
- Only the non-linear P3M limiter curve changes across the three box sizes; the dynamical, fixed-Δa and Hubble limiter branches are resolution-independent (except that the dynamical limiter also depends on particle dynamics).
- The Hubble limiter caps Δln a per Hubble time, consistent with the class-computed background.

## Limitations

- Among the limiters, only the non-linear P3M limiter depends on particle dynamics and hence on simulation resolution; the other branches are fixed functions of scale factor.
- A buggy time-stepping limiter for fluid components existed prior to the v0.2.0 release of CONCEPT; results reconstructed from older versions may be unreliable.
- CONCEPT 1.0 uses the standard slab-decomposition of FFTW, which limits the number of CPU cores participating in the long-range force computation — relevant if limiters are extracted from a live high-resolution run rather than reconstructed analytically.

## Evidence

- [full_text] task_005 finding: [evidence span withheld — anchor status: not_found]
- [full_text] task_005 workflow: [evidence span withheld — anchor status: not_found]
- [full_text] task_005 validation: [evidence span withheld — anchor status: not_found]
- [abstract] article abstract: "designed to agree with that of linear Einstein-Boltzmann codes such as class (Blas, Lesgourgues & Tram 2011)"
