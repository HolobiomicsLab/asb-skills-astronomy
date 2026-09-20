---
name: halo-particle-boundness-classification
description: Use when an N-body/hydro snapshot (e.g. HDF5 particle data) contains
  halo candidates whose membership needs a physically motivated bound/unbound split,
  or when testing whether bound and unbound halo populations are distinct components.
when_to_use_negative:
- When a pre-existing halo catalogue already provides physically motivated membership
  and only masses/radii are needed — this skill is for re-deriving boundness from
  particle energies, not reading catalogues.
- When the analysis requires time-integrated particle trajectories or accretion history
  rather than instantaneous information — the Strawberry binding criterion deliberately
  uses only instantaneous potential information.
- When the data are not simulation particles (e.g. observational galaxy catalogues)
  — the criterion requires per-particle gravitational potential energies.
edam_operation: http://edamontology.org/operation_3436
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: strawberry halo finder
  role: Topology-based halo finder implementing the boosted-potential binding criterion
    that labels each particle as bound or unbound
  repo: github.com/trg-richardson/strawberry
- name: numpy
  role: Array computation for particle positions, velocities, and per-population diagnostic
    statistics
- name: scipy
  role: Scientific routines supporting density-profile and virial-ratio computations
- name: h5py
  role: Loading the example particle snapshot provided with the tools/ example codes
- name: tqdm
  role: Progress reporting during particle-level processing in the tools/ example
    codes
- name: hilbertcurve
  role: Space-filling-curve ordering of particles in the tools/ example codes
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202556846
    title: 'STRAWBERRY: Finding haloes in the gravitational potential'
  source_content_hash: sha256:0508ebce1097b0f8fc38e18a3c302153ca5c6df90c18b10cab5a75dbef57998d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - h5py
  - hilbertcurve
  - numpy
  - scipy
  - strawberry halo finder
  - tqdm
  edam_operation: http://edamontology.org/operation_3436
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# halo-particle-boundness-classification

## Summary

Classify every dark-matter particle in a cosmological snapshot as bound or unbound using the Strawberry halo finder's boosted-potential binding criterion, then quantify how the two populations differ in phase space, virial ratio, density profile, and redshift evolution to test the two-component picture of haloes.

## When to use

Use when an N-body/hydro snapshot (e.g. HDF5 particle data) contains halo candidates whose membership needs a physically motivated bound/unbound split, or when testing whether bound and unbound halo populations are distinct components.

## When NOT to use

- When a pre-existing halo catalogue already provides physically motivated membership and only masses/radii are needed — this skill is for re-deriving boundness from particle energies, not reading catalogues.
- When the analysis requires time-integrated particle trajectories or accretion history rather than instantaneous information — the Strawberry binding criterion deliberately uses only instantaneous potential information.
- When the data are not simulation particles (e.g. observational galaxy catalogues) — the criterion requires per-particle gravitational potential energies.

## Inputs

- Dark-matter particle snapshot in HDF5 format (e.g. the example snapshot shipped in the Strawberry repository's tools/ directory)
- Halo candidate regions whose particle membership is to be split into bound and unbound populations

## Outputs

- Per-particle bound/unbound labels from the Strawberry boosted-potential binding criterion
- Phase-space (position-velocity) distributions computed separately for bound and unbound populations
- Virial ratio (2K/|U|) per population, identifying which component is virialised
- Spherically averaged density profile per population with contrasting extents
- Population-level comparison table (bound_unbound_diagnostics.csv) with phase-space, virial, and density-profile diagnostics

## How to apply

Load the example particle snapshot from the repository's tools/ directory with h5py and install Strawberry via 'clone repository' + 'pip install .' (requires numpy, scipy, cython, setuptools and a c++ compiler; tools/ examples additionally need h5py, tqdm, hilbertcurve). Run the Strawberry finder with the boosted-potential binding criterion: a particle is bound if its energy does not exceed the escape energy set by the closest saddle point connecting to a deeper potential minimum — this uses the gravitational potential evaluated in an accelerated reference frame and includes the gravitational effect of all particles, with no ad hoc overdensity thresholds. Label each particle bound or unbound, then compute per-population diagnostics: the phase-space (position-velocity) distribution, the virial ratio 2K/|U|, and the spherically averaged density profile and its extent. Assemble these population-level diagnostics into a comparison table (e.g. bound_unbound_diagnostics.csv) and contrast the components: the bound component should be virialised and finite-extent, the unbound component un-virialised and rapidly evolving.

## Related tools

- **strawberry halo finder** (Topology-based halo finder implementing the boosted-potential binding criterion that labels each particle as bound or unbound) — github.com/trg-richardson/strawberry
- **numpy** (Array computation for particle positions, velocities, and per-population diagnostic statistics)
- **scipy** (Scientific routines supporting density-profile and virial-ratio computations)
- **h5py** (Loading the example particle snapshot provided with the tools/ example codes)
- **tqdm** (Progress reporting during particle-level processing in the tools/ example codes)
- **hilbertcurve** (Space-filling-curve ordering of particles in the tools/ example codes)

## Examples

```
git clone github.com/trg-richardson/strawberry; pip install .
```

## Evaluation signals

- The bound_unbound_diagnostics.csv table reports phase-space distribution, virial ratio, and density-profile diagnostics for BOTH populations.
- Bound component is virialised (2K/|U| ≈ 1) with a finite extent; unbound component is un-virialised with rapidly evolving, extended structure.
- The bound/unbound split shows very distinct phase-space distributions, matching the distinct characteristics reported in the paper.
- No ad hoc overdensity thresholds were introduced; binding decisions derive from the escape energy at the closest saddle point connecting to a deeper potential minimum.
- Redshift-evolution diagnostics reproduce the paper's finding that particles typically become bound between their first peri- and apo-centric passage.

## Limitations

- Requires a c++ compiler for the Cython-based installation ('pip install .'); this dependency is listed as a missing signal in the article's environment extraction.
- The criterion uses only instantaneous information and includes the gravitational effect of all particles — results depend on the snapshot's potential quality and particle resolution.
- Binding is decided per snapshot; the timing of when particles become bound (between first peri- and apo-centric passage) is inferred from redshift evolution, not from the single-snapshot classification itself.

## Evidence

- [methods] boosted-potential binding criterion (escape energy set by the closest saddle point connecting to a deeper potential minimum): [evidence span withheld — anchor status: not_found]
- [abstract] two distinct populations confirmed across four diagnostics: "bound and unbound populations show very distinct characteristics through their distribution in phase space, as well as in their density profiles, virial ratios, and redshift evolution"
- [abstract] no ad hoc thresholds; all particles contribute to binding: "it does not require any ad hoc thresholds (such as overdensity criteria), while it does include the gravitational effect of all particles in the binding criterion"
- [readme] installation path for the finder: "## Instalation

- clone repository
- `pip install .`"
- [readme] dependencies for the tools/ example workflow: "### Requirements for example codes in `tools/`

- h5py
- tqdm
- hilbertcurve"
