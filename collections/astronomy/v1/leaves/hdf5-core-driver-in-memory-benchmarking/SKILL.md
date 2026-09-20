---
name: hdf5-core-driver-in-memory-benchmarking
description: Use when you need to measure the single-core compute throughput of a
  chunked HDF5-based transform (e.g. rounding precision reduction) without confounding
  disk I/O, or to validate throughput claims against published MiB/s figures.
when_to_use_negative:
- You want end-to-end disk I/O performance numbers — the core driver stores data only
  in memory and will overestimate real-world storage throughput
- You are benchmarking the full two-step scheme including Bitshuffle lossless compression
  — this skill isolates the rounding-only ('Rounding') condition with no subsequent
  lossless compressor
- The dataset is much larger than available RAM, since the core driver holds the whole
  file in memory
tools:
- name: HDF5
  role: Storage layer for the benchmark; the 'core' file driver keeps datasets in
    memory and chunk shape (8, 8, 1024) sets the 512 KiB chunk granularity used for
    timing
- name: Bitshuffle
  role: Provides the authors' example precision-reduction (rounding) implementation
    and the HDF5 filter context in which throughput is measured; the Rounding condition
    excludes the compressor
  repo: https://github.com/kiyo-masui/bitshuffle
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.07.002
    title: A compression scheme for radio data in high performance computing
  source_content_hash: sha256:0f43d01ae1cadb31a429e55a274d925b18faff4952f08d8084bd380831989bdc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/kiyo-masui/bitshuffle
  tools:
  - Bitshuffle
  - HDF5
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# HDF5 core driver in-memory throughput benchmarking

## Summary

Benchmarks write/read throughput of a data-processing step (e.g. the paper's precision-reduction rounding) by storing visibility data via HDF5's 'core' file driver so I/O is to memory only, isolating compute throughput from disk effects. Used to reproduce the paper's reported ~300 MiB/s single-core precision-reduction throughput.

## When to use

Use when you need to measure the single-core compute throughput of a chunked HDF5-based transform (e.g. rounding precision reduction) without confounding disk I/O, or to validate throughput claims against published MiB/s figures.

## When NOT to use

- You want end-to-end disk I/O performance numbers — the core driver stores data only in memory and will overestimate real-world storage throughput
- You are benchmarking the full two-step scheme including Bitshuffle lossless compression — this skill isolates the rounding-only ('Rounding') condition with no subsequent lossless compressor
- The dataset is much larger than available RAM, since the core driver holds the whole file in memory

## Inputs

- Synthetic visibility dataset of shape (64, 136, 1024), each element a struct of two signed 32-bit little-endian integers (real and imaginary), totalling 68 MiB (numpy)
- HDF5 chunk shape (8, 8, 1024) producing 512 KiB chunks
- HDF5 'core' file driver configuration (backing-store disabled) via h5py

## Outputs

- Write throughput in MiB/s for the rounding-only precision-reduction condition
- Read throughput in MiB/s for the rounding-only precision-reduction condition
- Reproducibility check across repeated timing trials (within a few percent)

## How to apply

Generate a synthetic visibility dataset matching the paper's shape (64, 136, 1024), where each element is a struct of two signed 32-bit little-endian integers (real, imaginary), totalling 68 MiB. Write it with h5py/HDF5 using the 'core' file driver so the dataset exists only in memory, using chunk shape (8, 8, 1024) to yield 512 KiB chunks. Run the transform — here the rounding step, which rounds each element to the largest possible power of two subject to the maximum granularity limit — on a single thread with no subsequent lossless compression (the paper's 'Rounding' condition of Table 2). Time write and read throughput in MiB/s across repeated trials and check reproducibility to within a few percent. Validate against the paper's rounding-only result of 289 MiB/s write and 2574 MiB/s read at a 100.0% compression ratio.

## Related tools

- **HDF5** (Storage layer for the benchmark; the 'core' file driver keeps datasets in memory and chunk shape (8, 8, 1024) sets the 512 KiB chunk granularity used for timing)
- **Bitshuffle** (Provides the authors' example precision-reduction (rounding) implementation and the HDF5 filter context in which throughput is measured; the Rounding condition excludes the compressor) — https://github.com/kiyo-masui/bitshuffle

## Examples

```
import h5py, numpy as np; f = h5py.File('bench.h5', 'w', driver='core'); d = f.create_dataset('vis', shape=(64,136,1024), dtype=np.dtype([('re','<i4'),('im','<i4')]), chunks=(8,8,1024))
```

## Evaluation signals

- Measured write throughput is consistent with the paper's rounding-only value of 289 MiB/s within the stated few-percent reproducibility
- Measured read throughput is consistent with the paper's 2574 MiB/s at 100.0% compression ratio
- Repeated timing trials agree to within a few percent
- Total dataset size is 68 MiB (64 x 136 x 1024 elements x 8 bytes), confirming the synthetic data matches the paper's dimensions
- Compression ratio is 100.0% (no lossless compression applied in the Rounding condition)

## Limitations

- Throughput figures are processor-dependent: Bitshuffle achieves roughly 1 byte per clock cycle on modern x86, up to 2 bytes per clock cycle on Haswell (2013) and later, so absolute MiB/s will vary by machine
- The paper's reference numbers come from CHIME Pathfinder correlator data collected on January 25, 2015; synthetic data with different statistical properties may shift throughput and compression ratio
- The core driver benchmarks memory-resident I/O only and does not capture disk bandwidth effects

## Evidence

- [methods] Round all data to a multiple of the largest possible power of two subject to the limit on rounding granularity given in Equation 16.: [evidence span withheld — anchor status: not_found]
- [methods] Reference dataset provenance from the paper's comparisons: [evidence span withheld — anchor status: not_found]
