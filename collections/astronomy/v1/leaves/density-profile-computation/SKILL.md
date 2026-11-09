---
name: density-profile-computation
description: Use when a particle snapshot has been split into bound/unbound populations
  (e.g. by Strawberry's boosted-potential criterion) and you need radial density profiles
  to test which component is finite-extent and virialised.
when_to_use_negative:
- The particle data has not yet been split into bound/unbound populations — run the
  Strawberry halo finder with the boosted-potential criterion first, otherwise the
  'two populations' profiles are undefined
- The analysis goal is identifying haloes themselves (topology-based halo finding),
  not characterising the radial structure of already-labelled populations
- The data is an analytic or tabulated profile rather than discrete simulation particles,
  so a spherically averaged particle histogram is unnecessary
edam_operation: http://edamontology.org/operation_2945
tools:
- name: strawberry halo finder
  role: Produces the bound/unbound particle labels (boosted-potential, saddle-point
    escape energy) that define the two populations whose density profiles are contrasted
  repo: github.com/trg-richardson/strawberry
- name: h5py
  role: Loads the example particle snapshot from HDF5 into arrays for profile computation
- name: numpy
  role: Computes radial distances, radial binning, and the spherically averaged density
    profile per population
- name: scipy
  role: Supporting scientific-computation routines for the population diagnostics
    (listed as a requirement of the repository)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202556846
    title: 'STRAWBERRY: Finding haloes in the gravitational potential'
  source_content_hash: sha256:93e77c12aaba12b72778da1d65ce2592ef1563fb5e2e7a90823b436f90fc2f98
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - h5py
  - numpy
  - scipy
  - strawberry halo finder
  - tqdm
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# density-profile-computation

## Summary

Computes spherically averaged density profiles of particle populations (specifically Strawberry's bound vs unbound components, labelled via the boosted-potential saddle-point binding criterion) to contrast their radial extents. This demonstrates that haloes are two-component systems: the bound component is virialised with finite extent, while the unbound component is extended and un-virialised.

## When to use

Use when a particle snapshot has been split into bound/unbound populations (e.g. by Strawberry's boosted-potential criterion) and you need radial density profiles to test which component is finite-extent and virialised.

## When NOT to use

- The particle data has not yet been split into bound/unbound populations — run the Strawberry halo finder with the boosted-potential criterion first, otherwise the 'two populations' profiles are undefined
- The analysis goal is identifying haloes themselves (topology-based halo finding), not characterising the radial structure of already-labelled populations
- The data is an analytic or tabulated profile rather than discrete simulation particles, so a spherically averaged particle histogram is unnecessary

## Inputs

- Particle snapshot in HDF5 format (positions, velocities, masses), e.g. the example snapshot provided with the tools/ example codes
- Per-particle bound/unbound labels from the Strawberry halo finder's boosted-potential binding criterion

## Outputs

- Spherically averaged density profile rho(r) for the bound particle population
- Spherically averaged density profile rho(r) for the unbound particle population
- Population-level diagnostics table saved as bound_unbound_diagnostics.csv, including density-profile characteristics alongside phase-space and virial-ratio diagnostics

## How to apply

Load the particle snapshot (e.g. the example snapshot shipped with the tools/ example codes) using h5py, and take the per-particle bound/unbound labels from the Strawberry halo finder, whose binding criterion sets the escape energy at the closest saddle point connecting to a deeper potential minimum. For each population separately, compute particle distances from the halo centre, bin them radially, and build the spherically averaged density profile (e.g. rho(r) = shell mass / (4/3 pi (r_out^3 - r_in^3)) per radial bin, using numpy). Contrast the extents of the two profiles: the paper reports the bound component has a finite extent while the unbound component extends much further, consistent with it being un-virialised. Assemble the density-profile diagnostics together with phase-space and virial-ratio (2K/|U|) diagnostics into bound_unbound_diagnostics.csv for validation.

## Related tools

- **strawberry halo finder** (Produces the bound/unbound particle labels (boosted-potential, saddle-point escape energy) that define the two populations whose density profiles are contrasted) — github.com/trg-richardson/strawberry
- **h5py** (Loads the example particle snapshot from HDF5 into arrays for profile computation)
- **numpy** (Computes radial distances, radial binning, and the spherically averaged density profile per population)
- **scipy** (Supporting scientific-computation routines for the population diagnostics (listed as a requirement of the repository))

## Examples

```
import numpy as np; r = np.linalg.norm(pos - centre, axis=1); edges = np.logspace(np.log10(r.min()), np.log10(r.max()), 50); rho, _ = np.histogram(r, bins=edges, weights=mass); rho /= (4/3*np.pi*np.diff(edges**3))
```

## Evaluation signals

- The output table bound_unbound_diagnostics.csv contains density-profile diagnostics for both the bound and the unbound populations (not just one), as required by the workflow validation
- The bound-component profile shows a finite extent (falls off to near-zero at finite radius), while the unbound-component profile extends beyond it, matching the reported two-component picture
- Profiles are computed as spherically averaged quantities per radial bin (shell volume normalisation), so total mass is recovered when the profile is integrated over radius
- The bound population's profile is consistent with a virialised, quasi-equilibrium component (e.g. agreeing virial ratio near 2K/|U| = 1 for the bound component, un-virialised for the unbound component)
- Diagnostics show distinct characteristics between populations, as the paper reports for phase space, density profiles, virial ratios, and redshift evolution

## Limitations

- Strawberry's binding criterion uses only instantaneous information, so the derived profiles and population split are snapshot-specific and will evolve with redshift (the unbound component is rapidly evolving)
- The profiles are spherically averaged, which assumes approximate radial symmetry around the halo centre; substructure and asphericity are washed out
- The two-component interpretation rests on the boosted-potential binding criterion (escape energy set by the closest saddle point connecting to a deeper potential minimum); profiles computed with ad hoc overdensity-based population splits will not match the reported separation

## Evidence

- [abstract] bound and unbound populations show very distinct characteristics through their distribution in phase space, as well as in their density profiles, virial ratios, and redshift evolution: "bound and unbound populations show very distinct characteristics through their distribution in phase space, as well as in their density profiles, virial ratios, and redshift evolution"
- [abstract] understand haloes as two-component systems, with one component being bound, virialised, of a finite extent, and evolving slowly in quasi-equilibrium and the other component being unbound,: "understand haloes as two-component systems, with one component being bound, virialised, of a finite extent, and evolving slowly in quasi-equilibrium and the other component being unbound,"
- [methods] Compute the spherically averaged density profile of each population and contrast their extents.: [evidence span withheld — anchor status: not_found]
- [abstract] Particles are considered to be bound if their energy does not exceed the escape energy of a potential well, given by the closest saddle point that connects to a deeper potential minimum: "Particles are considered to be bound if their energy does not exceed the escape energy of a potential well, given by the closest saddle point that connects to a deeper potential minimum"
- [readme] This repository is where we're developping Strawberry, a topology based halo finder which defines dark matter haloes in cosmological simulations using the boosted potential: "This repository is where we're developping Strawberry, a topology based halo finder which defines dark matter haloes in cosmological simulations using the boosted potential"
