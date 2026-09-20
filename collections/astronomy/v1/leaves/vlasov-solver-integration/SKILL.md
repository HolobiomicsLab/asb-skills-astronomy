---
name: vlasov-solver-integration
description: Use when a full 3D-3V Vlasov grid is intractable (memory in the EiB range)
  and the distribution function must be stored and propagated only where phase-space
  density is non-negligible.
when_to_use_negative:
- The problem is adequately handled by a magnetohydrodynamic (MHD) fluid model — Vlasiator's
  hybrid-Vlasov approach exists precisely to describe multi-temperature plasmas and
  non-MHD processes that MHD cannot self-consistently capture.
- The phase-space density is broadly populated everywhere, so the f_min-based sparsity
  discards little and the bookkeeping overhead of blocks and buffer layers yields
  no savings.
- Only post-processing or visualization of existing Vlasiator VLSV output is needed
  — Analysator should be used instead of integrating sparse storage into the solver.
tools:
- name: Vlasiator
  role: Reference hybrid-Vlasov simulation whose sparse velocity-space representation
    and solver design are cross-checked and reproduced
  repo: https://github.com/fmihpc/vlasiator
- name: vlasiator
  role: Installable reference codebase used to verify velocity-block handling against
    the local solver implementation
  repo: https://github.com/fmihpc/vlasiator/wiki/Installing-Vlasiator
- name: analysator
  role: Python tooling for reading and analyzing Vlasiator VLSV output files when
    validating the sparse benchmark results
  repo: https://github.com/fmihpc/analysator
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s41115-018-0003-2
    title: Vlasov methods in space physics and astrophysics
  source_content_hash: sha256:f00a5961f27ea503b7c4b8f7210778a09ddecf6c57b49fa760719d5268b16139
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fmihpc/vlasiator
  tools:
  - Vlasiator
  - analysator
  - vlasiator
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# vlasov-solver-integration

## Summary

Integrates a sparse velocity-space representation into a hybrid-Vlasov solver (Vlasiator) by storing and propagating the ion distribution function only where phase-space density is non-negligible, cutting memory and compute demand of a full Eulerian 3D-3V magnetospheric run from ~4 EiB to a feasible footprint.

## When to use

Use when a full 3D-3V Vlasov grid is intractable (memory in the EiB range) and the distribution function must be stored and propagated only where phase-space density is non-negligible.

## When NOT to use

- The problem is adequately handled by a magnetohydrodynamic (MHD) fluid model — Vlasiator's hybrid-Vlasov approach exists precisely to describe multi-temperature plasmas and non-MHD processes that MHD cannot self-consistently capture.
- The phase-space density is broadly populated everywhere, so the f_min-based sparsity discards little and the bookkeeping overhead of blocks and buffer layers yields no savings.
- Only post-processing or visualization of existing Vlasiator VLSV output is needed — Analysator should be used instead of integrating sparse storage into the solver.

## Inputs

- task_001 solver code with full velocity-block handling
- Reference hybrid-Vlasov implementation from github.com/fmihpc/vlasiator
- f_min phase-space density threshold and buffer-layer width parameters
- Magnetospheric benchmark configuration (~10^18 cells / 4 EiB full grid)

## Outputs

- sparse_velocity_storage.py module integrated into the solver
- sparse_storage_benchmark.md with stored sample-point counts, memory footprint, and measured reduction factor (expected between 10^2 and 10^3 per Sects. 5.2/5.5)

## How to apply

First cross-check the existing velocity-block handling in the task_001 solver against the reference implementation in the fmihpc/vlasiator repository. Then implement the sparse representation: discard velocity cells whose phase-space density falls below the f_min threshold, retain a buffer layer of marginally populated cells around surviving regions, and repack the remaining cells into fixed 4x4x4 velocity cell blocks. The buffer layer is essential because adjacent position- or velocity-space cells are retained to model propagation and acceleration accurately. Integrate the sparse-storage module so translation and acceleration steps operate on the stored blocks only, then run the magnetospheric benchmark (~10^18 cells / 4 EiB full grid), counting stored sample points and memory footprint against the equivalent full-grid representation.

## Related tools

- **Vlasiator** (Reference hybrid-Vlasov simulation whose sparse velocity-space representation and solver design are cross-checked and reproduced) — https://github.com/fmihpc/vlasiator
- **vlasiator** (Installable reference codebase used to verify velocity-block handling against the local solver implementation) — https://github.com/fmihpc/vlasiator/wiki/Installing-Vlasiator
- **analysator** (Python tooling for reading and analyzing Vlasiator VLSV output files when validating the sparse benchmark results) — https://github.com/fmihpc/analysator

## Examples

```
git clone --recurse-submodules https://github.com/fmihpc/vlasiator
```

## Evaluation signals

- The measured phase-space reduction factor falls between 10^2 and 10^3, as claimed in Sects. 5.2/5.5 of the paper.
- Velocity-block handling matches the reference implementation in the fmihpc/vlasiator repository on cross-check.
- Stored sample points and memory footprint are counted and reported against the equivalent full-grid (~10^18 cells / 4 EiB) representation in sparse_storage_benchmark.md.
- Translation and acceleration steps operate only on stored 4x4x4 velocity cell blocks, with no reads from discarded (below f_min) cells.
- A buffer layer of adjacent marginally populated cells is retained around surviving regions so propagation and acceleration remain accurate at the sparsity boundary.

## Limitations

- The sparse representation retains a buffer layer of adjacent position- or velocity-space cells, so savings are never the full ratio of discarded cells and depend on the f_min threshold choice.
- Vlasiator's computational challenges remain immense even with sparsity; the method requires massively parallel computations on tens of thousands of cores via advanced HPC techniques.
- Git submodules must be initialized correctly when cloning the reference code (git clone --recurse-submodules), otherwise dependent header libraries will be missing.

## Evidence

- [abstract] Vlasiator, the world’s first global hybrid-Vlasov simulation for the Earth’s magnetic domain, the magnetosphere.: "Vlasiator, the world’s first global hybrid-Vlasov simulation for the Earth’s magnetic domain, the magnetosphere."
- [results] the distribution function is stored and propagated only where its phase-space density is non-negligible: [evidence span withheld — anchor status: not_found]
- [readme] In Vlasiator, ions are represented as velocity distribution functions, while electrons are magnetohydrodynamic fluid: [evidence span withheld — anchor status: external_doc]
- [methods] We use advanced high performance computing techniques to allow massively parallel computations on tens of thousands of cores.: [evidence span withheld — anchor status: external_doc]
