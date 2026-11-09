---
name: nbody-simulation-parameter-configuration
description: 'Use when setting up or reproducing a CONCEPT 1.0 N-body run: you need
  to specify the ΛCDM background cosmology, particle count/box size, or time-stepping
  parameters and want the exact parameter files the authors used.'
when_to_use_negative:
- When performing linear-theory-only predictions — use CLASS directly instead of a
  configured N-body run
- 'When high-resolution (k_Nyquist > 5 Mpc/h) runs and runtime is the priority: the
  paper finds GADGET faster than CONCEPT at high resolution'
- When you only need code-to-code power-spectrum benchmarking against GADGET-2/GADGET-4
  rather than reproducing the paper's specific parameter setups
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3298
tools:
- name: CONCEPT 1.0
  role: N-body simulation code whose parameter files are configured and executed
  repo: github.com/jmd-dk/concept
- name: class
  role: linear Einstein–Boltzmann code whose output CONCEPT initial conditions are
    designed to agree with; bundled during installation
  repo: github.com/lesgourg/class_public
- name: Python
  role: language CONCEPT is written in; parameter files and post-processing scripts
    (NumPy, Matplotlib) are Python-based
- name: Docker
  role: provides prebuilt images of CONCEPT 1.0 for running configured simulations
    without manual dependency installation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/mnras/stac568
    title: The cosmological simulation code CO<i>N</i>CEPT 1.0
  source_content_hash: sha256:8927b36558c6492435916624f64b85aa124b30249a4dd5a207c94da8d5450cf2
schema_version: 0.2.0
metadata:
  license_tier: restricted
  tools:
  - CONCEPT 1.0
  - Docker
  - Matplotlib
  - NumPy
  - Python
  - class
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3298
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# nbody-simulation-parameter-configuration

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Configure CONCEPT 1.0 cosmological N-body simulations by loading parameter files from the public concept1.0-data repository, setting the ΛCDM cosmology, particle load, and box size so that published simulations (e.g. the Figure 2 time-step limiter study for L_box ∈ {2, 1, 1/2} × 3√N Mpc/h) can be reproduced or extended.

## When to use

Use when setting up or reproducing a CONCEPT 1.0 N-body run: you need to specify the ΛCDM background cosmology, particle count/box size, or time-stepping parameters and want the exact parameter files the authors used.

## When NOT to use

- When performing linear-theory-only predictions — use CLASS directly instead of a configured N-body run
- When high-resolution (k_Nyquist > 5 Mpc/h) runs and runtime is the priority: the paper finds GADGET faster than CONCEPT at high resolution
- When you only need code-to-code power-spectrum benchmarking against GADGET-2/GADGET-4 rather than reproducing the paper's specific parameter setups

## Inputs

- Simulation parameter files from the concept1.0-data repository (github.com/Aarhus Cosmology/concept1.0-data)
- ΛCDM cosmological parameters (Table 1 of the CONCEPT 1.0 paper)
- Choice of box size L_box ∈ {2, 1, 1/2} × 3√N Mpc/h and particle number N
- Optionally, linear transfer functions / background from CLASS for initial conditions

## Outputs

- Configured CONCEPT 1.0 parameter file defining cosmology, particle load, and time-stepping limiters
- CONCEPT simulation runs per box size (e.g. three configurations for the resolution study)
- Time-step / scale-factor trajectory data usable for Δln a(a) analysis

## How to apply

Retrieve the simulation parameter files from github.com/Aarhus Cosmology/concept1.0-data rather than authoring them from scratch, since these recreate the paper's published simulations. Set the Table 1 ΛCDM cosmology (consistent with linear Einstein–Boltzmann output from CLASS, which CONCEPT is designed to agree with) and the particle load appropriate to the chosen box size; for resolution studies, vary L_box over {2, 1, 1/2} × 3√N Mpc/h while keeping the particle number fixed so mean inter-particle separation scales with the box. Ensure the time-stepping configuration exposes the relevant limiters (dynamical displacement limiter, fixed-Δa branches, Hubble limiter, and non-linear P3M limiter) since only the P3M limiter depends on resolution. Run CONCEPT via the installed binary or the official Docker image (hub.docker.com/r/jmddk/concept), and validate the configuration by checking that outputs reproduce the expected limiter ordering (smaller boxes → smaller P3M-limited steps).

## Related tools

- **CONCEPT 1.0** (N-body simulation code whose parameter files are configured and executed) — github.com/jmd-dk/concept
- **class** (linear Einstein–Boltzmann code whose output CONCEPT initial conditions are designed to agree with; bundled during installation) — github.com/lesgourg/class_public
- **Python** (language CONCEPT is written in; parameter files and post-processing scripts (NumPy, Matplotlib) are Python-based)
- **Docker** (provides prebuilt images of CONCEPT 1.0 for running configured simulations without manual dependency installation)

## Examples

```
git clone https://github.com/Aarhus Cosmology/concept1.0-data && ./concept -c parameter_files/lcdm_box=2.dat -t basic
```

## Evaluation signals

- Configured run reproduces the paper's limiter structure: fixed-Δa branches transition near matter–Λ equality
- P3M-limiter time steps are monotonically ordered with box size: L_box = 2 > 1 > 1/2 × 3√N Mpc/h yields strictly larger allowed steps
- Linear-stage output agrees with CLASS to the level documented in the paper (the code is 'designed to agree with' CLASS)
- Parameter files loaded from concept1.0-data recreate the published simulations without modification
- Non-linear P3M limiter — the only dynamics-dependent limiter — responds to resolution changes while fixed-Δa and Hubble limiters do not

## Limitations

- Only the non-linear P3M limiter depends on particle dynamics and hence resolution; other limiters (fixed-Δa, Hubble) are resolution-independent
- A buggy time-stepping limiter for fluid components existed prior to the v0.2.0 fix — use version 1.0.x
- FFTW slab decomposition limits the number of CPU cores usable for the long-range force computation, constraining very large configurations
- CONCEPT is slower than GADGET for high-resolution simulations (k_Nyquist > 5 Mpc/h)

## Evidence

- [abstract] simulation parameter files for recreating the simulations producing this data — are made publicly available at github.com/Aarhus Cosmology/concept1.0-data: [evidence span withheld — anchor status: not_found]
