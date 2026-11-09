---
name: gravitational-potential-topology-saddle-point-analysis
description: Use when particles in a cosmological N-body snapshot (positions, velocities,
  masses) must be labelled bound vs unbound using only instantaneous information,
  without overdensity thresholds or iterative unbinding.
when_to_use_negative:
- The analysis requires overdensity-based halo definitions (e.g. FoF or spherical-overdensity
  thresholds) rather than the boosted-potential binding criterion — strawberry explicitly
  avoids ad hoc thresholds such as overdensity criteria.
- Only a subset of particles can be included in the potential calculation — the binding
  criterion's rationale requires the gravitational effect of all particles.
- Time-integrated or multi-snapshot orbit information is required — the criterion
  is deliberately instantaneous-only.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3473
tools:
- name: strawberry halo finder
  role: Computes the boosted potential and performs the topological identification
    of potential minima and closest saddle points for the binding criterion
  repo: github.com/trg-richardson/strawberry
- name: h5py
  role: Reads particle snapshot data and writes per-particle bound/unbound labels
    to HDF5 output
- name: numpy
  role: Dependency for the strawberry installation and numerical handling of particle
    arrays
- name: cython
  role: Build dependency for strawberry's Cython extensions installed via `pip install
    .`
- name: hilbertcurve
  role: Dependency for the repository's tools/ example codes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202556846
    title: 'STRAWBERRY: Finding haloes in the gravitational potential'
  source_content_hash: sha256:79be77c1ae6fd4fb7b54d84103c2382ed84c716bc70c1d646e2a4a252c5acb93
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - cython
  - h5py
  - hilbertcurve
  - numpy
  - scipy
  - strawberry halo finder
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# gravitational-potential-topology-saddle-point-analysis

## Summary

Classifies dark-matter particles as bound or unbound by computing the boosted potential (gravitational potential from an accelerated reference frame), identifying potential minima and the closest saddle points connecting to deeper minima, and comparing each particle's energy to the resulting escape energy — with no ad hoc thresholds such as overdensity criteria.

## When to use

Use when particles in a cosmological N-body snapshot (positions, velocities, masses) must be labelled bound vs unbound using only instantaneous information, without overdensity thresholds or iterative unbinding.

## When NOT to use

- The analysis requires overdensity-based halo definitions (e.g. FoF or spherical-overdensity thresholds) rather than the boosted-potential binding criterion — strawberry explicitly avoids ad hoc thresholds such as overdensity criteria.
- Only a subset of particles can be included in the potential calculation — the binding criterion's rationale requires the gravitational effect of all particles.
- Time-integrated or multi-snapshot orbit information is required — the criterion is deliberately instantaneous-only.

## Inputs

- Particle snapshot with positions, velocities, and masses (HDF5 readable with h5py, e.g. from the repository's tools/ example codes)

## Outputs

- Per-particle bound/unbound binding labels (e.g. particle_binding_labels.h5 written with h5py)
- Potential minima and their escape energies defined by closest saddle points connecting to deeper potential minima

## How to apply

Load a particle snapshot (positions, velocities, masses) from a snapshot file, e.g. the repository's tools/ example codes read HDF5 via h5py. Install the strawberry halo finder from the cloned repository with `pip install .`, which builds its Cython extensions with setuptools and a c++ compiler; the tools/ examples additionally need h5py, tqdm, and hilbertcurve. Use strawberry to compute the gravitational potential from an accelerated reference frame (the 'boosted potential'), which includes the gravitational effect of all particles in the binding criterion. Identify potential minima and the closest saddle point connecting each potential well to a deeper potential minimum to obtain the well's escape energy. Classify each particle as bound if its energy does not exceed the escape energy of that closest saddle point, using instantaneous information only, and write per-particle bound/unbound labels to an output file such as particle_binding_labels.h5 with h5py. Validate that every input particle received a label and that the bound and unbound populations are separable in phase space, density profiles, and virial ratios.

## Related tools

- **strawberry halo finder** (Computes the boosted potential and performs the topological identification of potential minima and closest saddle points for the binding criterion) — github.com/trg-richardson/strawberry
- **h5py** (Reads particle snapshot data and writes per-particle bound/unbound labels to HDF5 output)
- **numpy** (Dependency for the strawberry installation and numerical handling of particle arrays)
- **cython** (Build dependency for strawberry's Cython extensions installed via `pip install .`)
- **hilbertcurve** (Dependency for the repository's tools/ example codes)

## Examples

```
git clone github.com/trg-richardson/strawberry && cd strawberry && pip install .
```

## Evaluation signals

- Output file particle_binding_labels.h5 exists and contains a label for every input particle
- Bound and unbound populations are separable in phase-space distribution as the binding criterion implies
- Bound and unbound components show distinct density profiles and virial ratios, consistent with a two-component halo picture (bound, virialised, quasi-equilibrium vs unbound, un-virialised)
- The criterion can be checked to use no ad hoc thresholds (no overdensity criteria) and only instantaneous particle information
- Bound fraction should be consistent with particles typically becoming bound between their first peri- and apo-centric passage

## Limitations

- The binding criterion is instantaneous-only; it does not use particle histories or multi-snapshot information
- Installation requires building Cython extensions with setuptools and a c++ compiler, so a toolchain failure will block the workflow
- The repository README lists h5py, tqdm, and hilbertcurve as requirements only for the example codes in tools/, not the core install

## Evidence

- [abstract] criterion: "Particles are considered to be bound if their energy does not exceed the escape energy of a potential well, given by the closest saddle point that connects to a deeper potential minimum"
- [abstract] boosted potential: [evidence span withheld — anchor status: not_found]
- [abstract] no ad hoc thresholds: "it does not require any ad hoc thresholds (such as overdensity criteria), while it does include the gravitational effect of all particles in the binding criterion"
- [readme] install: "- clone repository
- `pip install .`"
- [abstract] phase space: "bound and unbound populations show very distinct characteristics through their distribution in phase space, as well as in their density profiles, virial ratios, and redshift evolution"
