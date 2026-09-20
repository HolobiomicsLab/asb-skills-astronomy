---
name: cython-numeric-kernel-implementation
description: Use when a pure-Python/NumPy loop over N-body particle pairs (e.g. within
  a short-range cut-off x_r) is the bottleneck and per-pair work must be compiled
  to C while retaining Python-level orchestration.
when_to_use_negative:
- The workload is already vectorisable with NumPy or covered by an existing library
  (e.g. FFTW-based long-range P3M mesh computation) rather than an irregular pair
  loop
- You need to modify high-level simulation logic or component behaviour rather than
  a hot inner loop — CONCEPT keeps such logic in Python
- The target platform lacks a C compiler toolchain (CONCEPT targets Linux with a full
  build chain of zlib, GSL, MPI, FFTW, Python)
tools:
- name: Cython
  role: Translates the Python-implemented kernel to equivalent C code for compiled-speed
    pair traversal
  repo: https://github.com/jmd-dk/concept
- name: NumPy
  role: Generates the synthetic uniform random particle distribution and handles array-level
    precomputation around the kernel
  repo: https://github.com/numpy/numpy
- name: Python
  role: Host language for orchestration; CONCEPT is the first massively parallel cosmological
    simulation code written in Python
  repo: https://github.com/jmd-dk/concept
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/mnras/stac568
    title: The cosmological simulation code CO<i>N</i>CEPT 1.0
  source_content_hash: sha256:6a660caf579a5dc2d48e713158e8b8758307b83fc65bd97b905d54bdce940566
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
yes_rate: 0.75
n_claims: 12
---

# cython-numeric-kernel-implementation

> **License: restricted** — no clear open-source license detected for the underlying tool; verify licensing before commercial use or redistribution. <!-- asb-license-banner -->
## Summary

Implement performance-critical numerical kernels (e.g. short-range pair-force traversal with minimum-image periodic distances and early rejection) as Cython code, following CONCEPT 1.0's approach of translating Python-level simulation logic to equivalent C code while keeping the orchestration in Python/NumPy.

## When to use

Use when a pure-Python/NumPy loop over N-body particle pairs (e.g. within a short-range cut-off x_r) is the bottleneck and per-pair work must be compiled to C while retaining Python-level orchestration.

## When NOT to use

- The workload is already vectorisable with NumPy or covered by an existing library (e.g. FFTW-based long-range P3M mesh computation) rather than an irregular pair loop
- You need to modify high-level simulation logic or component behaviour rather than a hot inner loop — CONCEPT keeps such logic in Python
- The target platform lacks a C compiler toolchain (CONCEPT targets Linux with a full build chain of zlib, GSL, MPI, FFTW, Python)

## Inputs

- NumPy array of particle positions (~10^5 particles) in a periodic box, generated with a fixed random seed
- Short-range cut-off x_r (float)
- Tile/subtile pair tables covering all pairs with separation <= x_r

## Outputs

- Cython-compiled kernel computing short-range pair forces with minimum-image periodic distances
- Tally of visited, rejected same-tile, and rejected neighbour-tile candidate pairs (tile_rejection_report.csv)

## How to apply

Isolate the inner numerical loop (here: pair traversal over tiles/subtiles of extent >= x_r in every dimension) into a Cython function with typed memoryviews; declare primitive types (cdef double) and disable bounds checking for the hot loop. Compute minimum-image periodic distances inside the kernel and early-reject candidate pairs that lie in non-adjacent (sub)tiles or exceed the cut-off x_r, mirroring CONCEPT's 9%/66%/91% rejection of candidates in same-tile, face-sharing, and edge-only configurations. Keep pair-table construction (tile/subtile pair tables covering every pair with separation <= x_r) in Python/NumPy, and precompute that bookkeeping before invoking the Cython kernel. Compile with Cython, which 'translates Python code to equivalent C code', and combine with NumPy/FFTW for the surrounding numerics.

## Related tools

- **Cython** (Translates the Python-implemented kernel to equivalent C code for compiled-speed pair traversal) — https://github.com/jmd-dk/concept
- **NumPy** (Generates the synthetic uniform random particle distribution and handles array-level precomputation around the kernel) — https://github.com/numpy/numpy
- **Python** (Host language for orchestration; CONCEPT is the first massively parallel cosmological simulation code written in Python) — https://github.com/jmd-dk/concept

## Examples

```
python -c "import numpy as np; np.random.seed(42); pos=np.random.uniform(0,100,(10**5,3)); np.save('pos.npy',pos)"  # then cythonize the pair kernel over tiles of size >= x_r
```

## Evaluation signals

- Brute-force O(N^2) completeness check on a small subsample finds zero pairs with true separation <= x_r missed by the tiling/kernel
- Rejection fractions of candidate pairs fall in the ranges reported for ideal cubic tiles of volume x_r^3: ~9% same-tile, ~66% face-sharing, ~91% edge-only early rejections
- Kernel compiles cleanly via Cython to C and runs with bounded/index-unsafe flags without memory errors
- Per-pair minimum-image distances agree with a NumPy reference implementation to floating-point tolerance
- Fractions written to tile_rejection_report.csv reconcile with the tallied visited/rejected counts

## Limitations

- Cython kernels shine on irregular pair loops; for structured mesh work CONCEPT instead delegates to FFTW, and pure NumPy/FFTW use alone limits achievable performance
- Early-rejection statistics depend on the tile size choice: tiles must be at least x_r per dimension; deviating from cubic tiles of volume x_r^3 changes the rejection fractions
- CONCEPT 1.0's own P3M implementation is noted as inefficient, so the Cython short-range kernel is not a blanket guarantee of overall performance parity with GADGET at high resolutions

## Evidence

- [abstract] performance to some extent is obtainable through the use of numerical libraries such as NumPy (Harris et al. 2020) and FFTW: [evidence span withheld — anchor status: not_found]
