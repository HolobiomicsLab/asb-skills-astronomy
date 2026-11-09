---
name: periodic-box-domain-decomposition
description: 'Use when implementing or verifying the short-range pair traversal of
  a P3M-style N-body solver in a periodic box: you need a tiling scheme that guarantees
  completeness for all pairs within the cut-off x_r.'
when_to_use_negative:
- The solver has no short-range cut-off component (e.g. pure long-range PM or particle-mesh-only
  computation) — tiling for x_r completeness is pointless
- Boundary conditions are non-periodic, so minimum-image distances used in the Cython
  traversal kernel are invalid
- Input positions are already partitioned or paired by another method (e.g. a tree
  code) and re-tiling would be redundant
tools:
- name: CONCEPT 1.0
  role: Reference P3M implementation whose tiles-and-subtiles short-range pairing
    scheme (section 2.1.3) is being reconstructed
  repo: https://github.com/jmd-dk/concept
- name: Cython
  role: Language for the pair-traversal kernel with minimum-image periodic distances
    and early rejection of non-interacting candidates
- name: NumPy
  role: Generates the synthetic uniform random particle distribution inside the periodic
    box used to validate the tiling
  repo: https://github.com/numpy/numpy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/mnras/stac568
    title: The cosmological simulation code CO<i>N</i>CEPT 1.0
  source_content_hash: sha256:1e12beff4fc9f7c6df399154dc36a0773fa4168f091c1e61b93e5569dd708027
schema_version: 0.2.0
metadata:
  license_tier: restricted
  repo_url: https://github.com/jmd-dk/concept
  tools:
  - CONCEPT 1.0
  - Cython
  - NumPy
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# periodic-box-domain-decomposition

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Decomposes a periodic simulation box into spatial domains, tiles, and subtiles — with tile extents of at least the short-range cut-off x_r in every dimension — so that the P3M short-range force computation visits every particle pair with separation <= x_r while early-rejecting non-interacting candidates.

## When to use

Use when implementing or verifying the short-range pair traversal of a P3M-style N-body solver in a periodic box: you need a tiling scheme that guarantees completeness for all pairs within the cut-off x_r.

## When NOT to use

- The solver has no short-range cut-off component (e.g. pure long-range PM or particle-mesh-only computation) — tiling for x_r completeness is pointless
- Boundary conditions are non-periodic, so minimum-image distances used in the Cython traversal kernel are invalid
- Input positions are already partitioned or paired by another method (e.g. a tree code) and re-tiling would be redundant

## Inputs

- Particle positions (NumPy array, e.g. ~10^5 synthetic uniform random particles in a periodic box, fixed random seed)
- Short-range cut-off radius x_r
- Tile/subtile geometry parameters (tiles per dimension, subtile subdivision, domain decomposition)

## Outputs

- Tile and subtile pair tables covering every particle pair with separation <= x_r within each domain
- Counts of visited, rejected same-tile, and rejected neighbour-tile candidate pairs
- tile_rejection_report.csv containing rejected same-tile and neighbour-tile candidate-pair fractions

## How to apply

Partition the periodic box into spatial domains, then within each domain build tiles whose extent in every dimension is at least the short-range cut-off x_r, subdividing further into subtiles and precomputing the tile/subtile pair tables so every pair with separation <= x_r is covered by a same-tile or neighbour-tile pairing. Implement the pair traversal as a Cython kernel that computes minimum-image periodic distances and early-rejects candidate pairs lying in non-adjacent (sub)tiles or beyond x_r. Validate completeness on a small subsample with a brute-force O(N^2) check, confirming zero pairs with true separation <= x_r are missed. For ideal cubic tiles of volume x_r^3, expect non-interacting candidate pairs to be early-rejected in roughly 9% of same-tile cases, 66% of face-sharing cases, and 91% of edge-only cases; tally visited/rejected counts over a synthetic uniform distribution (~10^5 particles, fixed seed) and write the fractions to tile_rejection_report.csv.

## Related tools

- **CONCEPT 1.0** (Reference P3M implementation whose tiles-and-subtiles short-range pairing scheme (section 2.1.3) is being reconstructed) — https://github.com/jmd-dk/concept
- **Cython** (Language for the pair-traversal kernel with minimum-image periodic distances and early rejection of non-interacting candidates)
- **NumPy** (Generates the synthetic uniform random particle distribution inside the periodic box used to validate the tiling) — https://github.com/numpy/numpy

## Examples

```
np.random.seed(0); pos = np.random.uniform(0.0, L, (100000, 3))
```

## Evaluation signals

- Brute-force O(N^2) completeness check on a small subsample finds zero pairs with true separation <= x_r missed by the tiling
- Every tile extent is at least x_r in each dimension of the box
- Rejection fractions for ideal cubic tiles of volume x_r^3 fall near the reported 9% (same-tile), 66% (face-sharing), and 91% (edge-only)
- Visited + rejected same-tile + rejected neighbour-tile candidate counts sum to the total number of candidate pairs traversed
- tile_rejection_report.csv exists and contains both same-tile and neighbour-tile candidate-pair fractions

## Limitations

- The completeness guarantee holds only when tile extent is at least x_r per dimension; smaller tiles break the pairing guarantee
- Rejection statistics are reported for ideal cubic tiles of volume x_r^3; non-cubic tile shapes will shift the fractions
- P3M is noted as inefficient within CONCEPT's discussion, and load imbalance in strongly clustered systems limits short-range/long-range performance

## Evidence

- [other] Tiles of size at least x_r per dimension ensure all interacting pairs are visited: [evidence span withheld — anchor status: not_found]
- [abstract] performance to some extent is obtainable through the use of numerical libraries such as NumPy (Harris et al. 2020) and FFTW: [evidence span withheld — anchor status: not_found]
