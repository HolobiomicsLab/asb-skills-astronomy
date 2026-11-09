---
name: memory-footprint-benchmarking
description: 'Reference for benchmarking a sparse-grid or adaptive phase-space representation
  against its dense counterpart: you have a full-grid memory estimate (e.g. ~4 EiB
  for Vlasiator''s 3D–3V magnetospheric configuration) and need a measured reduction
  factor.'
when_to_use_negative:
- Input is a particle-in-cell or Monte-Carlo (noise-carrying) representation with
  no Eulerian velocity grid to sparsify — the sparse block storage and f_min cutoff
  do not apply
- The simulation domain is small enough that the full 3D–3V distribution fits in memory,
  in which case sparsification overhead buys nothing and the 4 EiB-scale reduction
  framing is irrelevant
- You need physical validation results (verification against observations) rather
  than computational cost quantification — this skill measures footprint, not physics
  accuracy
tools:
- name: Vlasiator
  role: Global hybrid-Vlasov simulation whose sparse velocity-space representation
    is the subject of the memory-footprint benchmark; reference implementation for
    cross-checking velocity-block handling
  repo: https://github.com/fmihpc/vlasiator
- name: Analysator
  role: Python toolset for reading and analyzing Vlasiator VLSV output files, usable
    to inspect stored distribution-function data when auditing the benchmark
  repo: https://github.com/fmihpc/analysator
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s41115-018-0003-2
    title: Vlasov methods in space physics and astrophysics
  source_content_hash: sha256:8bad74fffe96695607bc9d20a1410517886e18d278925e446c3eac012f7ef2a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fmihpc/vlasiator
  tools:
  - Analysator
  - Vlasiator
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# memory-footprint-benchmarking

## Summary

Quantifies the memory and computational savings of a sparse Eulerian phase-space representation by counting stored sample points against the equivalent full-grid baseline, as Vlasiator does when storing the ion distribution function only where phase-space density is non-negligible.

## When to use

Reference for benchmarking a sparse-grid or adaptive phase-space representation against its dense counterpart: you have a full-grid memory estimate (e.g. ~4 EiB for Vlasiator's 3D–3V magnetospheric configuration) and need a measured reduction factor.

## When NOT to use

- Input is a particle-in-cell or Monte-Carlo (noise-carrying) representation with no Eulerian velocity grid to sparsify — the sparse block storage and f_min cutoff do not apply
- The simulation domain is small enough that the full 3D–3V distribution fits in memory, in which case sparsification overhead buys nothing and the 4 EiB-scale reduction framing is irrelevant
- You need physical validation results (verification against observations) rather than computational cost quantification — this skill measures footprint, not physics accuracy

## Inputs

- Task solver code with velocity-block handling (e.g. task_001 solver, cross-checked against fmihpc/vlasiator reference implementation)
- Reference implementation of sparse velocity-space representation from fmihpc/vlasiator
- Stated benchmark configuration (magnetospheric setup with ~10^18 full-grid cells / 4 EiB baseline)
- Phase-space density threshold f_min and buffer-layer width parameters

## Outputs

- sparse_velocity_storage.py sparse-storage module integrated into the solver
- sparse_storage_benchmark.md reporting stored sample points, memory footprint, and measured reduction factor
- Reduction factor (expected in the 10^2–10^3 range)

## How to apply

Start from the dense-baseline estimate: for Vlasiator's full 3D–3V Eulerian hybrid-Vlasov grid this is ~10^18 cells / 4 EiB. Implement the sparse representation by discarding velocity cells whose phase-space density falls below the f_min threshold, retaining a buffer layer of marginally populated cells around surviving regions so translation and acceleration can be modeled accurately, and repacking remaining cells into fixed 4×4×4 velocity-cell blocks. Integrate the sparse-storage module into the solver so propagation acts only on stored blocks. Run the stated benchmark configuration, counting stored sample points and memory footprint of the sparse run against the equivalent full-grid representation. Compute the reduction factor and verify it lies in the 10^2–10^3 range claimed in Sects. 5.2/5.5 of the Vlasiator review, writing results to a benchmark report alongside the storage module.

## Related tools

- **Vlasiator** (Global hybrid-Vlasov simulation whose sparse velocity-space representation is the subject of the memory-footprint benchmark; reference implementation for cross-checking velocity-block handling) — https://github.com/fmihpc/vlasiator
- **Analysator** (Python toolset for reading and analyzing Vlasiator VLSV output files, usable to inspect stored distribution-function data when auditing the benchmark) — https://github.com/fmihpc/analysator

## Examples

```
git clone --recurse-submodules https://github.com/fmihpc/vlasiator && python sparse_velocity_storage.py --f_min <threshold> --block 4x4x4 --output sparse_storage_benchmark.md
```

## Evaluation signals

- Measured reduction factor falls between 10^2 and 10^3, matching the claims in Sects. 5.2/5.5 of the Vlasiator review
- Sparse baseline denominator matches the stated full-grid estimate of ~10^18 cells / 4 EiB for the magnetospheric configuration
- Velocity cells below the f_min threshold are absent from storage, while a buffer layer of marginally populated cells surrounds each surviving region
- Remaining cells are repacked into fixed 4×4×4 velocity-cell blocks, and translation/acceleration steps touch only stored blocks
- Results (including the reduction factor) are written to sparse_storage_benchmark.md alongside sparse_velocity_storage.py

## Limitations

- The buffer layer of adjacent position- or velocity-space cells is retained for propagation/accuracy, so the true reduction is always less than discarding all low-density cells
- Results are configuration-dependent: the 10^2–10^3 reduction applies to the stated magnetospheric benchmark; other geometries or f_min choices will shift the factor
- The 4 EiB figure is a minimum dense-grid estimate; actual dense runs are infeasible, so comparison is against a computed baseline rather than an executed full-grid run

## Evidence

- [results] Sparse storage retains only populated regions: [evidence span withheld — anchor status: not_found]
- [results] Dense-grid baseline scale: [evidence span withheld — anchor status: not_found]
- [abstract] Vlasiator as the benchmark target: "A particular application of the Vlasov system is Vlasiator, the world’s first global hybrid-Vlasov simulation for the Earth’s magnetic domain, the magnetosphere."
- [readme] HPC necessity for the benchmark configuration: [evidence span withheld — anchor status: external_doc]
- [readme] Reference implementation acquisition: [evidence span withheld — anchor status: external_doc]
