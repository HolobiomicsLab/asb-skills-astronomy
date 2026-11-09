---
name: phase-space-density-thresholding
description: Use when an Eulerian Vlasov or hybrid-Vlasov solver's full 3D–3V velocity
  grid is prohibitively large (e.g. ~10^18 cells / 4 EiB) and only a small region
  of velocity space carries non-negligible phase-space density.
when_to_use_negative:
- The distribution function fills velocity space nearly uniformly, so sparsity savings
  are negligible and full-grid storage is simpler
- The solver already uses a particle/trajectory-based (PIC) representation rather
  than an Eulerian velocity grid
- Physical processes depend sensitively on tail cells below the f_min cutoff that
  the buffer layer cannot capture (e.g. very low-density beams far from the core population)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: Vlasiator
  role: Global hybrid-Vlasov simulation whose sparse velocity-space representation
    (f_min thresholding, buffer layer, 4×4×4 blocks) is the method this skill reproduces
  repo: https://github.com/fmihpc/vlasiator
- name: vlasiator
  role: Reference implementation used to cross-check velocity-block handling in the
    solver code
  repo: https://github.com/fmihpc/vlasiator/wiki/Installing-Vlasiator
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s41115-018-0003-2
    title: Vlasov methods in space physics and astrophysics
  source_content_hash: sha256:4b3a6335e0657d3e603885f98e11ac5372db476097721d047ec1b95decbda49d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fmihpc/vlasiator
  tools:
  - Vlasiator
  - vlasiator
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# phase-space-density-thresholding

## Summary

Sparse velocity-space representation used by Vlasiator: the distribution function is stored and propagated only where phase-space density is non-negligible (above an f_min threshold, with a buffer layer of marginally populated cells), reducing memory from ~4 EiB for a full 3D–3V Eulerian grid by factors of 10^2–10^3.

## When to use

Use when an Eulerian Vlasov or hybrid-Vlasov solver's full 3D–3V velocity grid is prohibitively large (e.g. ~10^18 cells / 4 EiB) and only a small region of velocity space carries non-negligible phase-space density.

## When NOT to use

- The distribution function fills velocity space nearly uniformly, so sparsity savings are negligible and full-grid storage is simpler
- The solver already uses a particle/trajectory-based (PIC) representation rather than an Eulerian velocity grid
- Physical processes depend sensitively on tail cells below the f_min cutoff that the buffer layer cannot capture (e.g. very low-density beams far from the core population)

## Inputs

- Solver code for the Eulerian 3D–3V hybrid-Vlasov distribution function (e.g. task_001 solver code)
- Velocity-space grid with per-cell phase-space density values f
- f_min threshold parameter defining non-negligible phase-space density
- Reference implementation: fmihpc/vlasiator repository for velocity-block handling

## Outputs

- Sparse-storage module (e.g. sparse_velocity_storage.py) storing only above-threshold cells plus buffer layer as 4×4×4 velocity cell blocks
- Benchmark report (e.g. sparse_storage_benchmark.md) with stored sample-point counts, memory footprint, and measured reduction factor vs. full grid
- Sparse solver in which translation and acceleration operate on stored blocks only

## How to apply

Inspect the velocity-space grid of the solver (e.g. a Vlasiator-style hybrid-Vlasov code) and discard velocity cells whose phase-space density f falls below an f_min threshold, since they carry negligible physical content. Retain a buffer layer of adjacent marginally populated position- or velocity-space cells so that propagation and acceleration steps remain accurate as distribution content moves between cells. Repack the surviving cells into fixed 4×4×4 velocity cell blocks so translation and acceleration operators act only on stored blocks, not the full grid. Benchmark against the equivalent full-grid representation by counting stored sample points and memory footprint; a correct implementation yields a reduction factor in the 10^2–10^3 range reported in Vlasiator Sects. 5.2/5.5. Cross-check block handling against the reference implementation in the fmihpc/vlasiator repository.

## Related tools

- **Vlasiator** (Global hybrid-Vlasov simulation whose sparse velocity-space representation (f_min thresholding, buffer layer, 4×4×4 blocks) is the method this skill reproduces) — https://github.com/fmihpc/vlasiator
- **vlasiator** (Reference implementation used to cross-check velocity-block handling in the solver code) — https://github.com/fmihpc/vlasiator/wiki/Installing-Vlasiator

## Examples

```
git clone --recurse-submodules https://github.com/fmihpc/vlasiator
```

## Evaluation signals

- Measured memory/point reduction factor versus the equivalent full-grid representation falls between 10^2 and 10^3, as claimed in Sects. 5.2/5.5
- No velocity cell with phase-space density above f_min is discarded; every surviving region is surrounded by the required buffer layer of adjacent cells
- Remaining cells are packed into fixed 4×4×4 velocity cell blocks and translation/acceleration steps touch only stored blocks
- Sparse results are numerically consistent with a full-grid reference run (conservation of distribution-function content through translation and acceleration)
- Benchmark report documents stored sample points and memory footprint against the ~10^18-cell / 4 EiB full-grid baseline

## Limitations

- A buffer layer must be retained around surviving regions; without it, propagation and acceleration errors appear at sparse-region boundaries
- Savings depend entirely on the sparsity of the distribution: the ~4 EiB estimate applies to the magnetospheric configuration without sparsity, and thresholds (f_min) introduce a tunable accuracy/memory trade-off
- Computational challenges remain immense; sparse storage alone does not remove the need for massively parallel HPC techniques (tens of thousands of cores)

## Evidence

- [results] Sparse representation saves memory: [evidence span withheld — anchor status: not_found]
- [results] Without sparsity the full grid needs ~4 EiB: [evidence span withheld — anchor status: not_found]
- [methods] Reference implementation for cross-checking: "https://github.com/fmihpc/vlasiator/wiki/Installing-Vlasiator"
- [readme] HPC context from README: [evidence span withheld — anchor status: external_doc]
