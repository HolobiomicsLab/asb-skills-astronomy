---
name: dark-matter-particle-binding-energy-assessment
description: Use when you have a cosmological simulation particle snapshot (positions,
  velocities, masses) and need a threshold-free per-particle bound/unbound classification
  for halo structure assignment.
when_to_use_negative:
- When an overdensity-threshold-based halo definition is explicitly desired (e.g.
  Friends-of-Friends with a fixed overdensity criterion) — strawberry's binding criterion
  deliberately avoids such ad hoc thresholds.
- When you need time-integrated binding history rather than an instantaneous classification
  — the strawberry criterion uses only instantaneous information.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3293
tools:
- name: strawberry halo finder
  role: computes the boosted potential, identifies potential minima and closest saddle
    points, and applies the escape-energy binding criterion
  repo: github.com/trg-richardson/strawberry
- name: h5py
  role: reads the input particle snapshot and writes the per-particle bound/unbound
    labels to particle_binding_labels.h5
- name: numpy
  role: numerical arrays for particle positions, velocities, and masses
- name: scipy
  role: dependency of the strawberry halo finder installation
- name: cython
  role: builds strawberry's Cython extensions during `pip install .`
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202556846
    title: 'STRAWBERRY: Finding haloes in the gravitational potential'
  source_content_hash: sha256:1e639376380962d699b8a561788daf89485312eb1309a743d61a5be33b2ff3f7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - cython
  - h5py
  - numpy
  - scipy
  - strawberry halo finder
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3293
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# dark-matter-particle-binding-energy-assessment

## Summary

Classifies individual dark matter particles as bound or unbound by comparing each particle's energy against the escape energy of its potential well, computed from the boosted potential (gravitational potential in an accelerated reference frame) using the strawberry halo finder. This criterion relies on no ad hoc thresholds and includes the gravitational effect of all particles, using instantaneous information only.

## When to use

Use when you have a cosmological simulation particle snapshot (positions, velocities, masses) and need a threshold-free per-particle bound/unbound classification for halo structure assignment.

## When NOT to use

- When an overdensity-threshold-based halo definition is explicitly desired (e.g. Friends-of-Friends with a fixed overdensity criterion) — strawberry's binding criterion deliberately avoids such ad hoc thresholds.
- When you need time-integrated binding history rather than an instantaneous classification — the strawberry criterion uses only instantaneous information.

## Inputs

- cosmological simulation particle snapshot (positions, velocities, masses), e.g. stored as HDF5 and read with h5py
- strawberry halo finder installation (numpy, scipy, cython, setuptools, c++ compiler)

## Outputs

- per-particle bound/unbound labels written to particle_binding_labels.h5

## How to apply

Load the particle snapshot (positions, velocities, masses) from the repository's tools/ example codes with h5py, then install strawberry with `pip install .` (it builds Cython extensions via setuptools and requires a c++ compiler). Use strawberry to compute the gravitational potential from an accelerated reference frame (the 'boosted potential'), which includes the gravitational effect of all particles. Identify potential minima and the closest saddle points connecting to deeper potential minima to obtain the escape energy of each potential well. Classify each particle as bound if its energy does not exceed the escape energy of that closest saddle point, and write the per-particle bound/unbound labels to particle_binding_labels.h5 with h5py. The rationale is that this binding criterion requires no ad hoc thresholds (such as overdensity criteria) and uses only instantaneous information.

## Related tools

- **strawberry halo finder** (computes the boosted potential, identifies potential minima and closest saddle points, and applies the escape-energy binding criterion) — github.com/trg-richardson/strawberry
- **h5py** (reads the input particle snapshot and writes the per-particle bound/unbound labels to particle_binding_labels.h5)
- **numpy** (numerical arrays for particle positions, velocities, and masses)
- **scipy** (dependency of the strawberry halo finder installation)
- **cython** (builds strawberry's Cython extensions during `pip install .`)

## Examples

```
pip install . ; python -c "import h5py; f=h5py.File('particle_binding_labels.h5','r'); print(f['labels'][:])"
```

## Evaluation signals

- particle_binding_labels.h5 exists and contains a label for every input particle
- the bound and unbound populations are separable in phase space, as the algorithm's binding criterion implies
- bound and unbound populations show distinct density profiles, virial ratios, and redshift evolution
- no ad hoc thresholds (such as overdensity criteria) were introduced anywhere in the classification pipeline

## Limitations

- The criterion classifies each particle using instantaneous information only, so it does not capture time-integrated binding history.
- Requires building Cython extensions via setuptools and a c++ compiler, so a working compilation toolchain is needed.
- Example codes in tools/ additionally require h5py, tqdm, and hilbertcurve.

## Evidence

- [abstract] binding criterion uses closest saddle point escape energy: "Particles are considered to be bound if their energy does not exceed the escape energy of a potential well, given by the closest saddle point that connects to a deeper potential minimum"
- [abstract] no ad hoc thresholds, all-particle gravity, instantaneous: "it does not require any ad hoc thresholds (such as overdensity criteria), while it does include the gravitational effect of all particles in the binding criterion"
- [abstract] boosted potential definition: [evidence span withheld — anchor status: not_found]
- [readme] README requirements for installation: "- numpy
- scipy
- cython
- setuptools
- c++ compiler"
- [abstract] phase-space distinctness of bound and unbound populations: "bound and unbound populations show very distinct characteristics through their distribution in phase space, as well as in their density profiles, virial ratios, and redshift evolution"
