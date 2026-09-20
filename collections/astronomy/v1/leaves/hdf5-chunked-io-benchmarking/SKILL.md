---
name: hdf5-chunked-io-benchmarking
description: Use when you need to quantify I/O read performance differences between
  FITS and HDF5 (chunked, optionally bitshuffle-compressed) for full-file vs. random
  slice access on large 3D scientific arrays.
when_to_use_negative:
- Full characterization of data-access-pattern effects on read performance is out
  of scope; use only for the two access patterns benchmarked (full-file and z-axis
  slice reads)
- The question involves lossy compression comparisons — the paper explicitly did not
  compare lossy compression algorithms
- The data are small enough to fit in cache, since cache-busting via fresh file copies
  would dominate the measurement
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: fits2hdf
  role: Converts the FITS file and produces HDFITS (HDF5) equivalents of the 3D dataset
    used for the read benchmark
  repo: https://github.com/telegraphic/fits2hdf
- name: h5py
  role: Pythonic interface used to write chunked HDF5 datasets (chunk shape 1000x10x10)
    and to time slice reads
- name: bitshuffle
  role: HDF5 filter (filter number 32008) applied to produce the bitshuffle-compressed
    HDF5 variant for compressed read benchmarking
  repo: https://github.com/kiyo-masui/bitshuffle
- name: HDF5
  role: Underlying binary data format providing chunked storage, B-tree chunk indexing,
    and filter pipelines exploited by the benchmark
- name: Python
  role: Language for the timing harness (numpy + h5py) that runs the 16-repetition,
    cache-busted read loops
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.05.001
    title: 'HDFITS: Porting the FITS data model to HDF5'
  source_content_hash: sha256:fc276fddb58437a3fb33d95020a01e239327f10bfa9819014271df7c61bef263
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/telegraphic/fits2hdf
  tools:
  - HDF5
  - Python
  - bitshuffle
  - fits2hdf
  - h5py
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# hdf5-chunked-io-benchmarking

## Summary

Benchmarking methodology from the HDFITS paper for measuring and comparing read performance of FITS files against chunked HDF5 (uncompressed and bitshuffle-compressed) for full-file reads and small z-axis slice reads of large 3D arrays.

## When to use

Use when you need to quantify I/O read performance differences between FITS and HDF5 (chunked, optionally bitshuffle-compressed) for full-file vs. random slice access on large 3D scientific arrays.

## When NOT to use

- Full characterization of data-access-pattern effects on read performance is out of scope; use only for the two access patterns benchmarked (full-file and z-axis slice reads)
- The question involves lossy compression comparisons — the paper explicitly did not compare lossy compression algorithms
- The data are small enough to fit in cache, since cache-busting via fresh file copies would dominate the measurement

## Inputs

- 3D numpy array (e.g. (10000, 200, 200) int32) produced by a prior data-generation step
- Serialized copies of the array: a FITS file, an uncompressed chunked HDF5 file, and a bitshuffle-compressed HDF5 file

## Outputs

- io_benchmark_results.csv with per-format full-read and slice-read timings (aggregated over 16 repetitions)
- Comparison table of FITS vs chunked HDF5 vs bitshuffle HDF5 read times

## How to apply

Serialize the same 3D array (e.g. the paper's (10000, 200, 200) int32 volume) three ways with fits2hdf/h5py: as FITS, as chunked uncompressed HDF5 (chunk shape 1000x10x10 in the paper), and as bitshuffle-compressed HDF5. For each file, time both a full-file read and a read of a pair of randomly chosen z-axis slices, repeating each measurement 16 times and forcing every repetition through a fresh cache-busting copy of the file to defeat OS page-cache effects. Aggregate the per-format timings (median or mean over the 16 repetitions) into io_benchmark_results.csv. The rationale is that HDF5 chunking lets small slice reads touch only the relevant chunks, whereas FITS requires reading the full file, so slice-read speedups (the paper saw ~0.21-0.24 s vs 21.1 s) dwarf the comparable full-read times (~13.9-15.7 s).

## Related tools

- **fits2hdf** (Converts the FITS file and produces HDFITS (HDF5) equivalents of the 3D dataset used for the read benchmark) — https://github.com/telegraphic/fits2hdf
- **h5py** (Pythonic interface used to write chunked HDF5 datasets (chunk shape 1000x10x10) and to time slice reads)
- **bitshuffle** (HDF5 filter (filter number 32008) applied to produce the bitshuffle-compressed HDF5 variant for compressed read benchmarking) — https://github.com/kiyo-masui/bitshuffle
- **HDF5** (Underlying binary data format providing chunked storage, B-tree chunk indexing, and filter pipelines exploited by the benchmark)
- **Python** (Language for the timing harness (numpy + h5py) that runs the 16-repetition, cache-busted read loops)

## Examples

```
import h5py, numpy as np; f = h5py.File('data.h5','w'); f.create_dataset('DATA', data=arr, chunks=(1000,10,10), compression=32008)
```

## Evaluation signals

- Slice-read timings: uncompressed HDF5 ~0.24 s and bitshuffle HDF5 ~0.21 s versus FITS ~21.1 s — HDF5 slice reads should be orders of magnitude faster than FITS
- Full-file read timings should be comparable across formats (~14.3 s FITS, ~15.7 s chunked HDF5, ~13.9 s bitshuffle HDF5), within generous tolerance for hardware variation
- Each timing is the aggregate (median/mean) of 16 repetitions performed against fresh cache-busting copies of the file
- All three serialized files contain the identical (10000, 200, 200) int32 array, so differences reflect I/O format only
- Results are written to io_benchmark_results.csv with per-format full-read and slice-read columns

## Limitations

- Full characterization of the effect of data access pattern upon read performance is beyond the scope of the paper
- No comparison of lossy compression algorithms was performed
- Absolute timings vary with hardware, so validation uses generous tolerance around the reference values
- Reported numbers assume the OS page cache is bypassed via fresh file copies per repetition; without this, cached reads skew results

## Evidence

- [results] Reading a pair of randomly chosen z-axis slices took 21.1 s with FITS versus 0.24 s for uncompressed HDF5 and 0.21 s for bitshuffle-compressed HDF5: [evidence span withheld — anchor status: not_found]
- [readme] Chunked storage / B-tree mechanism: [evidence span withheld — anchor status: not_found]
