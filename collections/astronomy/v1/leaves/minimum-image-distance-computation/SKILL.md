---
name: minimum-image-distance-computation
description: Use when implementing or validating short-range gravity pair enumeration
  in a periodic cosmological N-body box, e.g. reproducing CONCEPT 1.0's tiles-and-subtiles
  P3M scheme and its early-rejection statistics.
when_to_use_negative:
- The domain is non-periodic or open-boundary, where minimum-image wrapping of separations
  is invalid
- The goal is the long-range (mesh/FFT) part of the P3M solver, which does not use
  explicit pair distances
- Particle data is already a processed analysis product (e.g. power spectra or snapshots)
  rather than raw particle positions in a box
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: CONCEPT 1.0
  role: Reference P3M implementation whose tiles-and-subtiles short-range pairing
    scheme this skill reconstructs
  repo: https://github.com/jmd-dk/concept
- name: Cython
  role: Implements the pair-traversal kernel with minimum-image periodic distances
    and early rejection
- name: NumPy
  role: Generates the synthetic uniform random particle distribution in the periodic
    box
  repo: https://github.com/numpy/numpy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/mnras/stac568
    title: The cosmological simulation code CO<i>N</i>CEPT 1.0
  source_content_hash: sha256:835918a83a7f438e58f256d03a35c2ed68465e32b46a9aec2399d49abf7ef021
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
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# minimum-image-distance-computation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Computes periodic-boundary pair distances in N-body short-range force kernels by wrapping separations into the primary box via minimum-image, combined with tile/subtile pair tables so all pairs within the short-range cut-off x_r are visited. This is the core distance geometry underlying CONCEPT 1.0's P3M solver.

## When to use

Use when implementing or validating short-range gravity pair enumeration in a periodic cosmological N-body box, e.g. reproducing CONCEPT 1.0's tiles-and-subtiles P3M scheme and its early-rejection statistics.

## When NOT to use

- The domain is non-periodic or open-boundary, where minimum-image wrapping of separations is invalid
- The goal is the long-range (mesh/FFT) part of the P3M solver, which does not use explicit pair distances
- Particle data is already a processed analysis product (e.g. power spectra or snapshots) rather than raw particle positions in a box

## Inputs

- Synthetic uniform random particle distribution (~10^5 particles) in a periodic box, generated with NumPy using a fixed random seed
- Short-range cut-off radius x_r
- Tile/subtile grid dimensions with per-dimension tile extent >= x_r
- Precomputed tile and subtile pair tables

## Outputs

- tile_rejection_report.csv with rejected same-tile and neighbour-tile candidate-pair fractions
- Counts of visited, rejected same-tile, and rejected neighbour-tile candidate pairs
- Brute-force O(N^2) completeness check confirming zero missed pairs with separation <= x_r

## How to apply

Partition the periodic simulation box into spatial domains and further into tiles whose extent in every dimension is at least the short-range cut-off x_r (ideally cubic tiles of volume x_r^3), then precompute tile/subtile pair tables so every particle pair with separation <= x_r is covered by a same-tile or neighbour-tile pairing. Traverse candidate pairs in a Cython kernel that computes minimum-image periodic distances and early-rejects pairs in non-adjacent (sub)tiles or beyond x_r; for ideal cubic tiles this rejects ~9% of same-tile candidates, ~66% of face-sharing candidates, and ~91% of edge-only candidates for a uniform distribution. Validate completeness by comparing against a brute-force O(N^2) distance check on a subsample, confirming that zero pairs with true separation <= x_r are missed. Tally visited, rejected same-tile, and rejected neighbour-tile candidate counts and export them to tile_rejection_report.csv.

## Related tools

- **CONCEPT 1.0** (Reference P3M implementation whose tiles-and-subtiles short-range pairing scheme this skill reconstructs) — https://github.com/jmd-dk/concept
- **Cython** (Implements the pair-traversal kernel with minimum-image periodic distances and early rejection)
- **NumPy** (Generates the synthetic uniform random particle distribution in the periodic box) — https://github.com/numpy/numpy

## Examples

```
import numpy as np; X = np.random.default_rng(42).uniform(0, box, (10**5, 3)); d = X[:,None,:] - X[None,:,:]; d -= box*np.round(d/box); r = np.sqrt((d**2).sum(-1))  # minimum-image distances
```

## Evaluation signals

- Brute-force O(N^2) completeness check on a subsample finds zero missed pairs with true separation <= x_r
- Same-tile early-rejection fraction for a uniform distribution is near ~9%
- Face-sharing neighbour-tile early-rejection fraction is near ~66% and edge-only near ~91% for ideal cubic tiles of volume x_r^3
- Every dimension of every tile has extent >= x_r, guaranteeing full cut-off coverage
- Rejected candidate-pair fractions in tile_rejection_report.csv are consistent between runs with different random seeds

## Limitations

- Early-rejection percentages are calibrated for uniform random distributions; strongly clustered systems change the rejection statistics and, per the paper, load imbalance in clustered systems is a known CONCEPT performance limitation
- CONCEPT's P3M implementation is explicitly noted as '(inefficient)' in the paper's release discussion
- Larger or non-cubic tiles lower early-rejection efficiency and increase the candidate-pair workload

## Evidence

- [methods] Tiles of size at least x_r per dimension ensure all interacting pairs are visited: [evidence span withheld — anchor status: not_found]
- [intro] CONCEPT is a free and open-source simulation code: [evidence span withheld — anchor status: not_found]
- [discussion] Load imbalance limitation: [evidence span withheld — anchor status: not_found]
