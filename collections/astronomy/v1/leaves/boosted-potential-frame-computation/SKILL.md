---
name: boosted-potential-frame-computation
description: Use when you have a cosmological simulation particle snapshot (positions,
  velocities, masses) and need the gravitational potential in an accelerated reference
  frame to classify particles as bound or unbound, or to define haloes by potential-well
  topology.
when_to_use_negative:
- You need halo definitions based on overdensity criteria (e.g. M200-style friends-of-friends
  thresholds) — STRAWBERRY's boosted-potential approach explicitly avoids ad hoc thresholds,
  so forcing density-threshold conventions onto its output is the wrong frame
- You only have a static-frame potential already computed by another code and want
  to reuse it — the binding criterion depends specifically on the accelerated (boosted)
  reference-frame potential, not a static one
- The task requires time-averaged or orbit-history information — the criterion uses
  only instantaneous information, so it cannot answer questions about a particle's
  long-term orbital fate
tools:
- name: strawberry halo finder
  role: Computes the boosted-potential gravitational field and its potential minima/saddle-point
    topology from the particle snapshot
  repo: github.com/trg-richardson/strawberry
- name: cython
  role: Build dependency; strawberry's Cython extensions are compiled during `pip
    install .`
  repo: github.com/trg-richardson/strawberry
- name: setuptools
  role: Build backend used by `pip install .` to compile the Cython extensions
- name: h5py
  role: Loads the example particle snapshot (positions, velocities, masses) from the
    tools/ example codes and writes per-particle binding labels
- name: numpy
  role: Numerical dependency for array handling in strawberry and its example codes
- name: scipy
  role: Numerical/scientific dependency of strawberry
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202556846
    title: 'STRAWBERRY: Finding haloes in the gravitational potential'
  source_content_hash: sha256:da7d261606dab3483534f5e1fca82d5151013c2ccfd9ddeedf0a003607a594aa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - cython
  - h5py
  - numpy
  - scipy
  - setuptools
  - strawberry halo finder
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# boosted-potential-frame-computation

## Summary

Computes the gravitational potential as seen from an accelerated reference frame ('boosted potential') using the strawberry halo finder, so that particle binding can be judged with all particles' gravitational effect included and no ad hoc thresholds. This is the core potential-calculation stage that STRAWBERRY's saddle-point binding criterion depends on.

## When to use

Use when you have a cosmological simulation particle snapshot (positions, velocities, masses) and need the gravitational potential in an accelerated reference frame to classify particles as bound or unbound, or to define haloes by potential-well topology.

## When NOT to use

- You need halo definitions based on overdensity criteria (e.g. M200-style friends-of-friends thresholds) — STRAWBERRY's boosted-potential approach explicitly avoids ad hoc thresholds, so forcing density-threshold conventions onto its output is the wrong frame
- You only have a static-frame potential already computed by another code and want to reuse it — the binding criterion depends specifically on the accelerated (boosted) reference-frame potential, not a static one
- The task requires time-averaged or orbit-history information — the criterion uses only instantaneous information, so it cannot answer questions about a particle's long-term orbital fate

## Inputs

- Particle snapshot with positions, velocities, and masses (e.g. an HDF5 file read with h5py, as in the repository's tools/ example codes)

## Outputs

- Gravitational potential computed from an accelerated reference frame (boosted potential) over the particle set
- Optionally, per-particle bound/unbound binding labels written to an h5 file (e.g. particle_binding_labels.h5) via the saddle-point escape-energy criterion

## How to apply

Clone the strawberry repository and install it with `pip install .`, which builds its Cython extensions via setuptools and requires a working c++ compiler (README requirements: numpy, scipy, cython, setuptools, c++ compiler). Load the particle snapshot (positions, velocities, masses) from the repository's tools/ example codes using h5py; the tools/ example codes additionally require h5py, tqdm, and hilbertcurve. Run strawberry to compute the boosted potential, i.e. the gravitational potential from an accelerated reference frame including the gravitational effect of all particles, rather than a static-frame potential. The rationale is that the accelerated-frame potential underpins STRAWBERRY's binding criterion: a particle is bound if its energy does not exceed the escape energy of the potential well given by the closest saddle point connecting to a deeper potential minimum, which requires no ad hoc thresholds (such as overdensity criteria) and uses only instantaneous information. Decision point: if downstream labelling is the goal, feed the computed potential into the saddle-point escape-energy classification and write per-particle bound/unbound labels to an h5 file such as particle_binding_labels.h5.

## Related tools

- **strawberry halo finder** (Computes the boosted-potential gravitational field and its potential minima/saddle-point topology from the particle snapshot) — github.com/trg-richardson/strawberry
- **cython** (Build dependency; strawberry's Cython extensions are compiled during `pip install .`) — github.com/trg-richardson/strawberry
- **setuptools** (Build backend used by `pip install .` to compile the Cython extensions)
- **h5py** (Loads the example particle snapshot (positions, velocities, masses) from the tools/ example codes and writes per-particle binding labels)
- **numpy** (Numerical dependency for array handling in strawberry and its example codes)
- **scipy** (Numerical/scientific dependency of strawberry)

## Examples

```
git clone https://github.com/trg-richardson/strawberry; cd strawberry; pip install .
```

## Evaluation signals

- The Cython extension build completes during `pip install .` with a c++ compiler available (install errors here mean the boosted-potential code cannot run)
- particle_binding_labels.h5 (or equivalent output) exists and contains a label for every input particle — no particle dropped
- Bound and unbound populations are separable in phase space, as the algorithm's binding criterion implies
- Bound and unbound populations show distinct density profiles and virial ratios, consistent with the paper's finding of a bound, virialised, quasi-equilibrium component versus an unbound, un-virialised component
- The classification reproduces the no-threshold property: bound/unbound assignment changes when all particles' gravitational contributions are included, and relies on no overdensity cutoff parameter

## Limitations

- Build requires a c++ compiler plus numpy, scipy, cython, and setuptools; the example codes in tools/ additionally require h5py, tqdm, and hilbertcurve — a missing compiler is a listed missing signal for the install step
- The binding criterion is instantaneous-only, so it cannot certify long-term boundedness; particles typically become bound between their first peri- and apo-centric passage
- The approach is tied to cosmological dark-matter halo finding via the boosted potential (Stücker et al. 2021); applying it outside that physical regime is unsupported by the article

## Evidence

- [abstract] Boosted-potential role in binding: [evidence span withheld — anchor status: not_found]
- [abstract] Saddle-point escape-energy criterion: "Particles are considered to be bound if their energy does not exceed the escape energy of a potential well, given by the closest saddle point that connects to a deeper potential minimum"
- [readme] Repository purpose and boosted potential: "This repository is where we're developping Strawberry, a topology based halo finder which defines dark matter haloes in cosmological simulations using the boosted potential"
- [readme] Install and build requirements: "- clone repository
- `pip install .`"
- [readme] Example-code extra requirements: "### Requirements for example codes in `tools/`

- h5py
- tqdm
- hilbertcurve"
