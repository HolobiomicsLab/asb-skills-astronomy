---
name: boosted-potential-binding-energy-evaluation
description: Use when every particle in a cosmological simulation snapshot must be
  labelled bound/unbound without overdensity thresholds, or when separating halo populations
  by phase-space distribution, virial ratio (2K/|U|), or density-profile extent.
when_to_use_negative:
- When a halo definition based on overdensity criteria is explicitly wanted — Strawberry's
  binding criterion intentionally requires no ad hoc thresholds such as overdensity
  criteria
- When the analysis needs time-integrated orbital history rather than a binding decision
  from instantaneous information only
- When the input is not a particle snapshot with positions/velocities (e.g. already
  aggregated catalogues or feature tables), since the criterion requires per-particle
  energies
tools:
- name: strawberry halo finder
  role: implements the boosted-potential binding criterion that labels each particle
    bound or unbound
  repo: github.com/trg-richardson/strawberry
- name: h5py
  role: loads the example particle snapshot from HDF5 in the tools/ example codes
- name: numpy
  role: numerical computation of population diagnostics (kinetic/potential energy
    terms, profiles)
- name: scipy
  role: numerical routines supporting the halo finder and diagnostics
- name: cython
  role: compiled implementation of the topology-based finder (installed via `pip install
    .`)
- name: tqdm
  role: progress reporting in the tools/ example codes
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202556846
    title: 'STRAWBERRY: Finding haloes in the gravitational potential'
  source_content_hash: sha256:498231eaf4a2208c13f21b163bad0bb4200cd90584baffd95464f426bb39c4b1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - cython
  - h5py
  - numpy
  - scipy
  - strawberry halo finder
  - tqdm
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# boosted-potential-binding-energy-evaluation

## Summary

Classifies every dark-matter particle in a simulation snapshot as bound or unbound using Strawberry's 'boosted potential' criterion — binding is judged against the escape energy set by the closest saddle point connecting to a deeper potential minimum, with no ad hoc overdensity thresholds. This enables testing whether haloes are two-component systems (bound, virialised vs. unbound, un-virialised).

## When to use

Use when every particle in a cosmological simulation snapshot must be labelled bound/unbound without overdensity thresholds, or when separating halo populations by phase-space distribution, virial ratio (2K/|U|), or density-profile extent.

## When NOT to use

- When a halo definition based on overdensity criteria is explicitly wanted — Strawberry's binding criterion intentionally requires no ad hoc thresholds such as overdensity criteria
- When the analysis needs time-integrated orbital history rather than a binding decision from instantaneous information only
- When the input is not a particle snapshot with positions/velocities (e.g. already aggregated catalogues or feature tables), since the criterion requires per-particle energies

## Inputs

- particle snapshot from a cosmological simulation (positions and velocities, read via h5py; example snapshots ship in the strawberry tools/ directory)
- gravitational potential computed in the boosted (accelerated) reference frame for the particle set

## Outputs

- per-particle bound/unbound labels assigned by the boosted-potential escape-energy criterion
- phase-space (position-velocity) distribution statistics for bound and unbound populations
- virial ratio (2K/|U|) for each population
- spherically averaged density profiles per population
- comparison table saved as bound_unbound_diagnostics.csv

## How to apply

Load the particle snapshot (HDF5, read with h5py) and install Strawberry from its repository with `pip install .` (Cython/c++ compiler required). Run the Strawberry halo finder with the boosted-potential binding criterion: the gravitational potential is evaluated from an accelerated reference frame, and a particle is considered bound if its energy does not exceed the escape energy of the potential well, given by the closest saddle point that connects to a deeper potential minimum; the gravitational effect of all particles enters the binding criterion and only instantaneous information is used. Split particles into the bound and unbound populations and, for each, compute the phase-space (position–velocity) distribution, the virial ratio 2K/|U|, and the spherically averaged density profile to contrast their extents. Assemble these population-level diagnostics into a comparison table saved as bound_unbound_diagnostics.csv. Accept the result if both populations are reported and show the paper's distinct characteristics: the bound component virialised with finite extent, the unbound component un-virialised.

## Related tools

- **strawberry halo finder** (implements the boosted-potential binding criterion that labels each particle bound or unbound) — github.com/trg-richardson/strawberry
- **h5py** (loads the example particle snapshot from HDF5 in the tools/ example codes)
- **numpy** (numerical computation of population diagnostics (kinetic/potential energy terms, profiles))
- **scipy** (numerical routines supporting the halo finder and diagnostics)
- **cython** (compiled implementation of the topology-based finder (installed via `pip install .`))
- **tqdm** (progress reporting in the tools/ example codes)

## Examples

```
git clone github.com/trg-richardson/strawberry && cd strawberry && pip install .
```

## Evaluation signals

- The diagnostics table reports phase-space distribution, virial ratio, and density-profile metrics for BOTH the bound and unbound populations
- Bound component is virialised (2K/|U| consistent with virial equilibrium) with a finite-extent density profile; unbound component is un-virialised and rapidly evolving
- No ad hoc overdensity threshold was introduced: binding is decided purely by the escape energy of the closest saddle point connecting to a deeper potential minimum
- Every particle in the snapshot receives exactly one bound/unbound label, and the gravitational contribution of all particles was included in the criterion
- Output file bound_unbound_diagnostics.csv exists and matches the population-level comparison schema

## Limitations

- Building Strawberry requires a c++ compiler plus cython/setuptools (`pip install .`); a missing c++ compiler is a documented setup gap
- The tools/ example codes additionally require h5py, tqdm, and hilbertcurve beyond the core requirements
- The criterion uses only instantaneous information, so it does not encode assembly history; the two-component interpretation (bound vs. unbound) is specific to haloes in cosmological simulations

## Evidence

- [abstract] Binding criterion defined by escape energy at closest saddle point: "Particles are considered to be bound if their energy does not exceed the escape energy of a potential well, given by the closest saddle point that connects to a deeper potential minimum"
- [abstract] Boosted potential discriminates bound from unbound: "a novel algorithm that discriminates between bound and unbound particles by consideration of the gravitational potential from an accelerated reference frame"
- [abstract] No ad hoc thresholds; all particles contribute: "it does not require any ad hoc thresholds (such as overdensity criteria), while it does include the gravitational effect of all particles in the binding criterion"
- [readme] Strawberry is a topology-based halo finder using the boosted potential: "This repository is where we're developping Strawberry, a topology based halo finder which defines dark matter haloes in cosmological simulations using the boosted potential"
