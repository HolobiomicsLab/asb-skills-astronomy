---
name: chunked-storage-io-profiling
description: Use when profiling storage I/O of chunked HDF5 visibility datasets under
  a rounding-only (precision-reduction, no lossless compression) condition, or when
  validating single-core compression/decompression throughput against the paper's
  Table 2 numbers.
when_to_use_negative:
- The pipeline includes the Bitshuffle lossless stage (e.g. Bitshuffle+LZ4 or Bitshuffle+ZSTD)
  — this skill profiles the rounding-only condition of Table 2, not the full two-step
  compression scheme
- Data elements are not adjacent highly-correlated typed binary values (e.g. uncorrelated
  floats); the bit-transpose/rearrangement rationale and throughput expectations from
  Bitshuffle would not transfer
- Multi-threaded/OpenMP throughput is the target — the paper's ~300 MiB/s figure is
  explicitly a single-core measurement
edam_operation: http://edamontology.org/operation_2409
tools:
- name: HDF5
  role: Storage layer for the synthetic visibility dataset; 'core' file driver keeps
    chunks in memory, chunk shape (8, 8, 1024) gives 512 KiB chunks for I/O timing
- name: Bitshuffle
  role: Source of the authors' example precision-reduction implementation (kiyo-masui/bitshuffle)
    whose single-core rounding throughput is being benchmarked; also provides the
    HDF5 filter (ID 32008) when the lossless stage is added later
  repo: https://github.com/kiyo-masui/bitshuffle
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.07.002
    title: A compression scheme for radio data in high performance computing
  source_content_hash: sha256:e4231c8dec741b089a8accf9ac9ace3c55622fe3fd63f190ff9923b4c8ad250c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/kiyo-masui/bitshuffle
  tools:
  - Bitshuffle
  - HDF5
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# chunked-storage-io-profiling

## Summary

Benchmark the read/write throughput of chunked HDF5 visibility data through a precision-reduction pipeline on a single core, to reproduce the paper's reported ~300 MiB/s rounding throughput (289 MiB/s write, 2574 MiB/s read).

## When to use

Use when profiling storage I/O of chunked HDF5 visibility datasets under a rounding-only (precision-reduction, no lossless compression) condition, or when validating single-core compression/decompression throughput against the paper's Table 2 numbers.

## When NOT to use

- The pipeline includes the Bitshuffle lossless stage (e.g. Bitshuffle+LZ4 or Bitshuffle+ZSTD) — this skill profiles the rounding-only condition of Table 2, not the full two-step compression scheme
- Data elements are not adjacent highly-correlated typed binary values (e.g. uncorrelated floats); the bit-transpose/rearrangement rationale and throughput expectations from Bitshuffle would not transfer
- Multi-threaded/OpenMP throughput is the target — the paper's ~300 MiB/s figure is explicitly a single-core measurement

## Inputs

- Synthetic visibility array of shape (64, 136, 1024), each element a struct of two signed 32-bit little-endian integers (real, imaginary), totalling 68 MiB
- HDF5 chunk configuration: chunk shape (8, 8, 1024) giving 512 KiB chunks, 'core' file driver (in-memory, no backing store)
- Rounding granularity limit from Equation 16 and acceptable fractional noise-variance increase f (recommended 10^-5 > f > 10^-2)

## Outputs

- Measured write throughput in MiB/s for the rounding-only condition on a single core
- Measured read throughput in MiB/s for the rounding-only condition on a single core
- Compression ratio (expected 100.0% for rounding-only, no lossless stage)
- Timing reproducibility report across repeated trials (few-percent spread)

## How to apply

Generate a synthetic visibility dataset of shape (64, 136, 1024) in which each element is a struct of two signed 32-bit little-endian integers (real, imaginary), totalling 68 MiB, using numpy — mirroring the CHIME Pathfinder correlator data collected 25 January 2015 used in the paper. Store it via HDF5 using the HDF5 'core' file driver so datasets exist only in memory, with chunk shape (8, 8, 1024) yielding 512 KiB chunks. Run the authors' lightly optimized precision-reduction implementation on a single thread, rounding elements to the largest possible power of two subject to the maximum granularity limit from Equation 16, with no subsequent Bitshuffle/LZ4 lossless compression (the 'Rounding' condition of Table 2). Time write and read throughputs in MiB/s across repeated trials; flag the run consistent if measured values match 289 MiB/s write and 2574 MiB/s read at 100.0% compression ratio within the paper's stated few-percent reproducibility.

## Related tools

- **HDF5** (Storage layer for the synthetic visibility dataset; 'core' file driver keeps chunks in memory, chunk shape (8, 8, 1024) gives 512 KiB chunks for I/O timing)
- **Bitshuffle** (Source of the authors' example precision-reduction implementation (kiyo-masui/bitshuffle) whose single-core rounding throughput is being benchmarked; also provides the HDF5 filter (ID 32008) when the lossless stage is added later) — https://github.com/kiyo-masui/bitshuffle

## Examples

```
import h5py, numpy as np; f = h5py.File('bench.hdf5', driver='core', backing_store=False); d = f.create_dataset('vis', shape=(64,136,1024), dtype=np.dtype('<i4,<i4'), chunks=(8,8,1024))
```

## Evaluation signals

- Measured write throughput falls within a few percent of the paper's 289 MiB/s rounding-only result at 100.0% compression ratio
- Measured read throughput falls within a few percent of the paper's 2574 MiB/s rounding-only result at 100.0% compression ratio
- Compression ratio reports exactly 100.0% (rounding only, no lossless compression applied)
- Repeated timing trials agree to within a few percent, confirming reproducibility
- Dataset totals 68 MiB with 512 KiB chunks (shape (8, 8, 1024) over a (64, 136, 1024) array of two-int32 structs), matching the paper's data configuration

## Limitations

- The ~300 MiB/s figure is for a lightly optimized example implementation on a single core of a modern processor; results vary with CPU generation (Bitshuffle README: ~1 byte/clock on modern x86, up to 2 bytes/clock on Haswell and later)
- The rounding step alone gives no compression (100.0% ratio); the 28% original-size compression cited for CHIME Pathfinder data requires the Bitshuffle lossless stage, which is out of scope here
- Benchmark uses in-memory HDF5 ('core' driver), so measured throughputs exclude physical disk effects; decompression in excess of 1 GB/s is achievable on a single core only for the full scheme

## Evidence

- [methods] Round all data to a multiple of the largest possible power of two subject to the limit on rounding granularity given in Equation 16.: [evidence span withheld — anchor status: not_found]
- [methods] The data used for the following comparisons was collected on January 25, 2015, starting at roughly 2:10AM PDT.: [evidence span withheld — anchor status: not_found]
