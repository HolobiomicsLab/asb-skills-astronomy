---
name: phase-space-distribution-analysis
description: Use when you have a cosmological simulation particle snapshot and need
  to test whether the bound and unbound material around a halo form physically distinct
  components in phase space, virial balance, and radial extent.
when_to_use_negative:
- You only need a halo catalogue via standard overdensity or friends-of-friends criteria
  — Strawberry's binding criterion deliberately avoids ad hoc thresholds such as overdensity
  criteria, so this skill answers a binding-energy separation question, not a halo-finding
  question
- Your input lacks particle-level positions and velocities (e.g. only a summary halo
  catalogue), since phase-space distributions and virial ratios require individual-particle
  instantaneous information
- You need orbital histories over time — the binding criterion uses only instantaneous
  information, not time-integrated trajectories
edam_operation: http://edamontology.org/operation_2945
tools:
- name: strawberry halo finder
  role: Runs the boosted-potential (accelerated reference frame) binding criterion
    to label every particle as bound or unbound
  repo: github.com/trg-richardson/strawberry
- name: h5py
  role: Loads the example particle snapshot provided with the tools/ example codes
- name: numpy
  role: Computes the phase-space distributions and virial ratio (2K/|U|) statistics
    per population
- name: scipy
  role: Supports the population-level distribution and density-profile computations
- name: tqdm
  role: Progress reporting for the example-code analysis loops
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202556846
    title: 'STRAWBERRY: Finding haloes in the gravitational potential'
  source_content_hash: sha256:fb4b4bf57239db6a860734e5759d20d8a285c58fb587d8678ce93ce34b36cca1
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
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# phase-space-distribution-analysis

## Summary

Separates halo particles into bound and unbound populations using Strawberry's boosted-potential binding criterion and contrasts them via phase-space distributions, virial ratios (2K/|U|), and spherically averaged density profiles, testing the two-component picture of haloes.

## When to use

Use when you have a cosmological simulation particle snapshot and need to test whether the bound and unbound material around a halo form physically distinct components in phase space, virial balance, and radial extent.

## When NOT to use

- You only need a halo catalogue via standard overdensity or friends-of-friends criteria — Strawberry's binding criterion deliberately avoids ad hoc thresholds such as overdensity criteria, so this skill answers a binding-energy separation question, not a halo-finding question
- Your input lacks particle-level positions and velocities (e.g. only a summary halo catalogue), since phase-space distributions and virial ratios require individual-particle instantaneous information
- You need orbital histories over time — the binding criterion uses only instantaneous information, not time-integrated trajectories

## Inputs

- Dark-matter particle snapshot (positions and velocities) in HDF5 format, loadable with h5py, e.g. the example particle snapshot provided with the Strawberry tools/ example codes
- Gravitational potential / boosted-potential information produced by the Strawberry halo finder for binding-energy assignment

## Outputs

- Per-particle bound/unbound labels from the boosted-potential binding criterion
- Separate phase-space (position-velocity) distributions for bound and unbound populations
- Virial ratios (2K/|U|) for each population
- Spherically averaged density profiles for each population
- Comparison table saved as bound_unbound_diagnostics.csv with phase-space, virial-ratio, and density-profile diagnostics for both populations

## How to apply

Load the example particle snapshot shipped with the Strawberry tools/ example codes using h5py, then run the Strawberry halo finder with the boosted-potential binding criterion — particles are bound if their energy does not exceed the escape energy set by the closest saddle point connecting to a deeper potential minimum, with no ad hoc overdensity thresholds. Label every particle as bound or unbound from this binding criterion, which includes the gravitational effect of all particles and uses only instantaneous information. Compute the position–velocity (phase-space) distribution separately for each population, then the virial ratio 2K/|U| for each to identify which component is virialised, and finally the spherically averaged density profile of each population to contrast their extents. Assemble the phase-space statistics, virial ratios, and density-profile characteristics into a comparison table and save it as bound_unbound_diagnostics.csv. Rationale: the paper shows the bound component is virialised, of finite extent, and slowly evolving in quasi-equilibrium, while the unbound component is un-virialised and rapidly evolving, so these diagnostics are exactly the discriminators of the two-component picture.

## Related tools

- **strawberry halo finder** (Runs the boosted-potential (accelerated reference frame) binding criterion to label every particle as bound or unbound) — github.com/trg-richardson/strawberry
- **h5py** (Loads the example particle snapshot provided with the tools/ example codes)
- **numpy** (Computes the phase-space distributions and virial ratio (2K/|U|) statistics per population)
- **scipy** (Supports the population-level distribution and density-profile computations)
- **tqdm** (Progress reporting for the example-code analysis loops)

## Examples

```
import h5py; f = h5py.File('snapshot.hdf5','r'); pos = f['PartType1/Pos'][:]; vel = f['PartType1/Vel'][:]  # then run Strawberry's boosted-potential binding on these particles
```

## Evaluation signals

- bound_unbound_diagnostics.csv reports phase-space distribution, virial-ratio, and density-profile diagnostics for BOTH populations (validation gate from the source card)
- Bound component is virialised (2K/|U| near unity) while the unbound component is un-virialised
- Bound population shows a finite-extent density profile, clearly contrasted in extent with the unbound population's profile
- Phase-space (position-velocity) distributions of the two populations are visibly and statistically distinct, matching the paper's finding of 'very distinct characteristics'
- Every particle in the snapshot carries exactly one bound/unbound label from the closest-saddle-point escape-energy criterion

## Limitations

- The binding criterion depends on Strawberry's boosted-potential computation and the closest-saddle-point escape energy; installing Strawberry requires a c++ compiler plus numpy, scipy, cython, and setuptools (flagged as a missing signal in the source card)
- The method uses only instantaneous information, so it cannot by itself recover particle orbital histories
- No ad hoc thresholds exist to tune — if the result disagrees with expectations, the failure is in the potential/saddle-point computation, not in a tunable cutoff

## Evidence

- [abstract] distinct phase-space and other diagnostics: "bound and unbound populations show very distinct characteristics through their distribution in phase space, as well as in their density profiles, virial ratios, and redshift evolution"
- [abstract] two-component picture: "one component being bound, virialised, of a finite extent, and evolving slowly in quasi-equilibrium"
- [abstract] binding criterion definition: "Particles are considered to be bound if their energy does not exceed the escape energy of a potential well, given by the closest saddle point that connects to a deeper potential minimum"
- [abstract] instantaneous, threshold-free criterion: "it does not require any ad hoc thresholds (such as overdensity criteria), while it does include the gravitational effect of all particles in the binding criterion"
- [readme] tool role: "a topology based halo finder which defines dark matter haloes in cosmological simulations using the boosted potential"
