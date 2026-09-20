---
name: halo-particle-phase-space-labeling
description: Use when you need per-particle bound/unbound labels for dark matter haloes
  in a cosmological N-body snapshot (positions, velocities, masses, e.g. HDF5 particle
  data) using only instantaneous information.
when_to_use_negative:
- The analysis requires time-resolved or trajectory-based binding (e.g. subhalo evolution
  over multiple snapshots) — this criterion uses instantaneous information only
- A simpler overdensity-based (e.g. spherical-overdensity) halo definition is sufficient,
  since this skill specifically avoids ad hoc thresholds like overdensity criteria
- The dataset is not a particle snapshot from a cosmological gravity simulation (the
  boosted-potential framework is defined for gravitational N-body/DM particle data)
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: strawberry halo finder
  role: Computes the boosted potential, finds potential minima and saddle points,
    and applies the binding criterion to label particles
  repo: github.com/trg-richardson/strawberry
- name: h5py
  role: Reads the input particle snapshot and writes the per-particle binding labels
    (particle_binding_labels.h5)
- name: numpy
  role: Array backend for particle data manipulation in the example codes in tools/
- name: scipy
  role: Listed requirement of the strawberry halo finder
- name: cython
  role: Builds the strawberry Cython extensions installed via `pip install .`
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202556846
    title: 'STRAWBERRY: Finding haloes in the gravitational potential'
  source_content_hash: sha256:671a85803e143d069ed1d484c7ba03c093a3d43e0f2cc3c3b4a0d6257500181d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - cython
  - h5py
  - numpy
  - scipy
  - strawberry halo finder
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# halo-particle-phase-space-labeling

## Summary

Classifies individual simulation particles as bound or unbound using STRAWBERRY's boosted-potential binding criterion: a particle is bound if its energy does not exceed the escape energy of the potential well defined by the closest saddle point connecting to a deeper potential minimum, with no ad hoc thresholds.

## When to use

Use when you need per-particle bound/unbound labels for dark matter haloes in a cosmological N-body snapshot (positions, velocities, masses, e.g. HDF5 particle data) using only instantaneous information.

## When NOT to use

- The analysis requires time-resolved or trajectory-based binding (e.g. subhalo evolution over multiple snapshots) — this criterion uses instantaneous information only
- A simpler overdensity-based (e.g. spherical-overdensity) halo definition is sufficient, since this skill specifically avoids ad hoc thresholds like overdensity criteria
- The dataset is not a particle snapshot from a cosmological gravity simulation (the boosted-potential framework is defined for gravitational N-body/DM particle data)

## Inputs

- Particle snapshot (positions, velocities, masses) from a cosmological simulation, stored in HDF5 (read with h5py)
- Gravitational potential field computed from the accelerated reference frame ('boosted potential')

## Outputs

- Per-particle bound/unbound classification labels written to particle_binding_labels.h5
- Set of potential minima and closest saddle points defining escape energies of potential wells

## How to apply

Load the particle snapshot (positions, velocities, masses) from the repository's tools/ example codes using h5py, then install the strawberry halo finder with `pip install .` (this builds Cython extensions via setuptools, requiring a C++ compiler). Compute the gravitational potential from an accelerated reference frame — the 'boosted potential' (Stücker et al. 2021) — which includes the gravitational effect of all particles rather than a traced subset. Identify potential minima and the closest saddle points connecting to deeper potential minima to obtain the escape energy of each potential well. Classify each particle as bound if its energy does not exceed the escape energy of that closest saddle point; the criterion uses instantaneous information only and requires no ad hoc thresholds such as overdensity criteria. Write the per-particle bound/unbound labels to an HDF5 file (particle_binding_labels.h5) with h5py.

## Related tools

- **strawberry halo finder** (Computes the boosted potential, finds potential minima and saddle points, and applies the binding criterion to label particles) — github.com/trg-richardson/strawberry
- **h5py** (Reads the input particle snapshot and writes the per-particle binding labels (particle_binding_labels.h5))
- **numpy** (Array backend for particle data manipulation in the example codes in tools/)
- **scipy** (Listed requirement of the strawberry halo finder)
- **cython** (Builds the strawberry Cython extensions installed via `pip install .`)

## Examples

```
pip install .  # then, with a snapshot loaded via h5py: labels = strawberry.classify_bound(positions, velocities, masses); write to particle_binding_labels.h5
```

## Evaluation signals

- particle_binding_labels.h5 exists and contains a label for every input particle (no unclassified particles)
- Bound and unbound populations are separable in phase space, as the binding criterion implies
- Bound and unbound components show distinct density profiles and virial ratios, consistent with a two-component halo picture (bound/virialised vs unbound/un-virialised)
- No ad hoc thresholds were needed: classification is driven purely by particle energy versus the saddle-point escape energy
- Installation succeeded with the Cython/C++ extension build (`pip install .` completed without compiler errors)

## Limitations

- The criterion uses only instantaneous information, so it cannot classify particles based on their orbital history
- The boosted-potential framework assumes an accelerated reference frame treatment (Stücker et al. 2021); results depend on this potential definition rather than a plain inertial-frame potential
- Building strawberry requires a C++ compiler, cython, and setuptools; example codes in tools/ additionally require h5py, tqdm, and hilbertcurve

## Evidence

- [abstract] Binding criterion relies on saddle-point escape energy: "Particles are considered to be bound if their energy does not exceed the escape energy of a potential well, given by the closest saddle point that connects to a deeper potential minimum"
- [abstract] No ad hoc thresholds; instantaneous information only: "it does not require any ad hoc thresholds (such as overdensity criteria), while it does include the gravitational effect of all particles in the binding criterion"
- [abstract] Boosted potential definition: [evidence span withheld — anchor status: not_found]
- [abstract] Distinct phase-space structure validates the split: "bound and unbound populations show very distinct characteristics through their distribution in phase space, as well as in their density profiles, virial ratios, and redshift evolution"
- [readme] Installation and build requirements: "This repository is where we're developping Strawberry, a topology based halo finder which defines dark matter haloes in cosmological simulations using the boosted potential"
- [readme] Tooling for example codes: "### Requirements for example codes in `tools/` - h5py - tqdm - hilbertcurve"
