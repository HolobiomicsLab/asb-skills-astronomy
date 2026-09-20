---
name: velocity-space-sparse-storage
description: Use when a full-grid Eulerian Vlasov or hybrid-Vlasov simulation is intractable
  in memory (e.g. ~10^18 cells / 4 EiB) and the distribution function is negligible
  in most velocity cells, so sparse velocity-space storage is needed.
when_to_use_negative:
- The distribution function has significant phase-space density in most velocity cells,
  so sparsity provides no saving and buffer-layer overhead dominates.
- The target is a fluid (MHD) closure where no velocity distribution function is stored
  at all.
- Only posterior analysis of existing .vlsv output files is needed — use Analysator/vlsv
  readers instead of modifying the solver's storage.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3928
tools:
- name: Vlasiator
  role: Reference global hybrid-Vlasov code whose sparse velocity-space representation
    and 4×4×4 velocity-block layout this skill reproduces and benchmarks
  repo: https://github.com/fmihpc/vlasiator
- name: vlasiator
  role: Repository cloned with --recurse-submodules following the Installing-Vlasiator
    wiki to cross-check velocity-block handling
  repo: https://github.com/fmihpc/vlasiator/wiki/Installing-Vlasiator
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1007/s41115-018-0003-2
    title: Vlasov methods in space physics and astrophysics
  source_content_hash: sha256:43618e2c655c48f9f496aebd278dc047961aa11ccec418bad1de01a04aff98ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fmihpc/vlasiator
  tools:
  - Vlasiator
  - vlasiator
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3928
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# velocity-space-sparse-storage

## Summary

Stores and propagates the Vlasiator distribution function only where phase-space density is non-negligible, using an f_min threshold, a buffer layer of marginally populated cells, and fixed 4×4×4 velocity cell blocks, reducing the memory footprint of a full Eulerian 3D–3V hybrid-Vlasov magnetospheric run (≥4 EiB without sparsity) by a factor of 10^2–10^3.

## When to use

Use when a full-grid Eulerian Vlasov or hybrid-Vlasov simulation is intractable in memory (e.g. ~10^18 cells / 4 EiB) and the distribution function is negligible in most velocity cells, so sparse velocity-space storage is needed.

## When NOT to use

- The distribution function has significant phase-space density in most velocity cells, so sparsity provides no saving and buffer-layer overhead dominates.
- The target is a fluid (MHD) closure where no velocity distribution function is stored at all.
- Only posterior analysis of existing .vlsv output files is needed — use Analysator/vlsv readers instead of modifying the solver's storage.

## Inputs

- Task_001 solver code for the hybrid-Vlasov velocity-block handling
- Reference fmihpc/vlasiator implementation (cloned with --recurse-submodules)
- Magnetospheric run configuration with baseline ~10^18 cells / 4 EiB full-grid equivalent
- Phase-space density threshold f_min for velocity-cell discarding

## Outputs

- sparse_velocity_storage.py sparse-storage module integrated into the solver
- sparse_storage_benchmark.md reporting stored sample points, memory footprint, and reduction factor
- Verified reduction factor between 10^2 and 10^3 relative to the full-grid representation

## How to apply

Cross-check the velocity-block handling of the base solver against the reference fmihpc/vlasiator implementation. Discard velocity cells whose phase-space density falls below the f_min threshold, retain a buffer layer of adjacent position- or velocity-space cells around surviving regions so propagation and acceleration are modelled accurately, and repack remaining cells into fixed 4×4×4 velocity cell blocks. Integrate the sparse-storage module so translation and acceleration steps operate on the stored blocks only, then benchmark the stated magnetospheric configuration (baseline ~10^18 cells / 4 EiB full grid) by counting stored sample points and memory footprint against the equivalent full-grid representation. Compute the measured reduction factor and verify it lies between 10^2 and 10^3 as claimed in Sects. 5.2/5.5. The rationale is that storing only non-negligible phase-space density yields large savings while the buffer layer preserves numerical accuracy at block boundaries.

## Related tools

- **Vlasiator** (Reference global hybrid-Vlasov code whose sparse velocity-space representation and 4×4×4 velocity-block layout this skill reproduces and benchmarks) — https://github.com/fmihpc/vlasiator
- **vlasiator** (Repository cloned with --recurse-submodules following the Installing-Vlasiator wiki to cross-check velocity-block handling) — https://github.com/fmihpc/vlasiator/wiki/Installing-Vlasiator

## Examples

```
git clone --recurse-submodules https://github.com/fmihpc/vlasiator && python sparse_velocity_storage.py --config magnetospheric_baseline.cfg --f_min 1e-15 --report sparse_storage_benchmark.md
```

## Evaluation signals

- Stored sample points and memory footprint are quantified against the equivalent full-grid representation (~10^18 cells / 4 EiB).
- Measured reduction factor falls between 10^2 and 10^3, matching the claim in Sects. 5.2/5.5.
- Velocity cells below f_min are discarded while a buffer layer of adjacent position- or velocity-space cells is retained around surviving regions.
- Remaining cells are packed into fixed 4×4×4 velocity cell blocks and translation/acceleration steps operate on stored blocks only.
- Downstream runs (e.g. two-population proton/O+ test) still produce valid .vlsv output with nonzero per-species moments.

## Limitations

- The buffer layer of adjacent cells must be retained; without it, propagation and acceleration across block boundaries are not modelled accurately.
- Even with sparsity, Vlasiator's computational challenges remain immense, requiring massively parallel computations on tens of thousands of cores.
- The benchmark reduction factor is configuration-specific (stated magnetospheric setup); other geometries or f_min choices may fall outside the 10^2–10^3 range.

## Evidence

- [methods] Sparse storage with f_min and buffer layer: [evidence span withheld — anchor status: not_found]
- [methods] Memory baseline without sparsity: [evidence span withheld — anchor status: not_found]
- [results] Expected reduction range: [evidence span withheld — anchor status: not_found]
- [readme] HPC techniques for massive parallelism: [evidence span withheld — anchor status: external_doc]
- [methods] Installing Vlasiator reference: "https://github.com/fmihpc/vlasiator/wiki/Installing-Vlasiator"
