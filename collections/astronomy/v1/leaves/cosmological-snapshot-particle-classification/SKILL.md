---
name: cosmological-snapshot-particle-classification
description: Use when a simulation snapshot (positions, velocities, masses) needs
  per-particle bound/unbound labels from a topology-based binding criterion that relies
  only on instantaneous information and no ad hoc thresholds.
when_to_use_negative:
- You need time-averaged or orbit-integrated binding information — the criterion uses
  only instantaneous information.
- You need subhalo structures defined by overdensity criteria (e.g. M200) rather than
  by escape energy of potential wells; strawberry deliberately avoids such thresholds.
- Your environment lacks a c++ compiler, so the Cython extensions cannot be built
  during `pip install .`.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: strawberry halo finder
  role: Computes the boosted potential, identifies potential minima and the closest
    saddle points connecting to deeper minima, and applies the escape-energy binding
    criterion to label particles bound or unbound.
  repo: github.com/trg-richardson/strawberry
- name: h5py
  role: Loads the input particle snapshot and writes the per-particle bound/unbound
    labels to particle_binding_labels.h5.
- name: cython
  role: Builds the compiled extensions installed by strawberry during `pip install
    .`.
- name: numpy
  role: Numerical handling of particle arrays during snapshot loading and label processing.
- name: scipy
  role: Numerical/analysis support in the binding-criterion workflow.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202556846
    title: 'STRAWBERRY: Finding haloes in the gravitational potential'
  source_content_hash: sha256:ad5b8ba0ddee7094088f90c1743b0431f627c5e66567ef14fb94637a31e710c4
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
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# cosmological-snapshot-particle-classification

## Summary

Classifies particles in a cosmological simulation snapshot as bound or unbound using strawberry's boosted-potential binding criterion: a particle is bound if its energy does not exceed the escape energy of the closest saddle point connecting to a deeper potential minimum. This avoids ad hoc thresholds (e.g. overdensity criteria) and includes the gravitational effect of all particles.

## When to use

Use when a simulation snapshot (positions, velocities, masses) needs per-particle bound/unbound labels from a topology-based binding criterion that relies only on instantaneous information and no ad hoc thresholds.

## When NOT to use

- You need time-averaged or orbit-integrated binding information — the criterion uses only instantaneous information.
- You need subhalo structures defined by overdensity criteria (e.g. M200) rather than by escape energy of potential wells; strawberry deliberately avoids such thresholds.
- Your environment lacks a c++ compiler, so the Cython extensions cannot be built during `pip install .`.

## Inputs

- Particle snapshot with positions, velocities, and masses (HDF5, e.g. from the repository's tools/ example codes)

## Outputs

- particle_binding_labels.h5 containing a per-particle bound/unbound label for every input particle

## How to apply

Load the particle snapshot (positions, velocities, masses) from the repository's tools/ example codes using h5py. Install strawberry with `pip install .`, which builds its Cython extensions via setuptools and requires a c++ compiler. Use strawberry to compute the gravitational potential from an accelerated reference frame (the boosted potential), including the gravitational effect of all particles. Identify potential minima and the closest saddle points connecting to deeper potential minima to obtain the escape energy of each potential well. Classify each particle as bound if its energy does not exceed the escape energy of that closest saddle point, using instantaneous information only, and write the per-particle labels to particle_binding_labels.h5 with h5py.

## Related tools

- **strawberry halo finder** (Computes the boosted potential, identifies potential minima and the closest saddle points connecting to deeper minima, and applies the escape-energy binding criterion to label particles bound or unbound.) — github.com/trg-richardson/strawberry
- **h5py** (Loads the input particle snapshot and writes the per-particle bound/unbound labels to particle_binding_labels.h5.)
- **cython** (Builds the compiled extensions installed by strawberry during `pip install .`.)
- **numpy** (Numerical handling of particle arrays during snapshot loading and label processing.)
- **scipy** (Numerical/analysis support in the binding-criterion workflow.)

## Examples

```
pip install .  # after cloning github.com/trg-richardson/strawberry; then run the tools/ example to write particle_binding_labels.h5 with h5py
```

## Evaluation signals

- particle_binding_labels.h5 exists and contains a label for every input particle.
- The bound and unbound populations are separable in phase space, as the binding criterion implies.
- Bound particles' energies do not exceed the escape energy of the closest saddle point; unbound particles' energies do.
- The bound component behaves as virialised and of finite extent, consistent with the article's finding that bound and unbound populations differ in density profiles, virial ratios, and redshift evolution.

## Limitations

- The criterion uses only instantaneous information, so particles are typically bound only between their first peri- and apocentric passage.
- Haloes are effectively two-component systems: the unbound component is un-virialised and rapidly evolving, so labels may change between snapshots.
- Installation requires a c++ compiler and Cython extension building; example codes additionally require h5py, tqdm, and hilbertcurve.

## Evidence

- [abstract] boosted potential binding criterion: [evidence span withheld — anchor status: not_found]
- [abstract] escape energy of closest saddle point: "Particles are considered to be bound if their energy does not exceed the escape energy of a potential well, given by the closest saddle point that connects to a deeper potential minimum"
- [abstract] no ad hoc thresholds, all particles included: "it does not require any ad hoc thresholds (such as overdensity criteria), while it does include the gravitational effect of all particles in the binding criterion"
- [readme] installation requires c++ compiler: "- cython
- setuptools
- c++ compiler"
- [readme] example codes require h5py: "### Requirements for example codes in `tools/`

- h5py
- tqdm
- hilbertcurve"
- [readme] repository defines strawberry as topology based halo finder: "This repository is where we're developping Strawberry, a topology based halo finder which defines dark matter haloes in cosmological simulations using the boosted potential"
