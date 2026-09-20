---
name: scientific-array-io-performance-analysis
description: Use when you must quantify I/O performance of large multi-dim scientific
  arrays stored in FITS vs chunked/compressed HDF5, especially for random z-axis slice
  reads vs full-file reads.
when_to_use_negative:
- You only need full-file (contiguous, whole-volume) read or write throughput — the
  paper found full reads comparable (14.3/15.7/13.9 s) and the decisive advantage
  lies in partial access patterns
- You need a full characterization of arbitrary data access patterns on read performance
  — the article states 'full characterization of the effect of data access pattern
  upon read performance is beyond the scope of this paper'
- You are evaluating lossy compression algorithms — the article explicitly notes 'we
  have not compared lossy compression algorithms'
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: fits2hdf
  role: Converts the array into FITS and HDFITS (HDF5) serializations used as benchmark
    subjects
  repo: https://github.com/telegraphic/fits2hdf
- name: h5py
  role: Python interface used to write chunked HDF5 datasets and time reads of full
    files and z-axis slices
  repo: https://github.com/telegraphic/fits2hdf
- name: bitshuffle
  role: HDF5 filter (filter number 32008) applied to the third serialization to test
    whether bit-level rearrangement compression affects read speed
  repo: https://github.com/kiyo-masui/bitshuffle
- name: Python
  role: Driver language for the benchmark loop, cache-busting copies, and aggregation
    of timings
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.05.001
    title: 'HDFITS: Porting the FITS data model to HDF5'
  source_content_hash: sha256:8dcbd42508e1265d86ccb44d5ce95ac7aaa30db46a588c9680fce183b2ce211d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/telegraphic/fits2hdf
  tools:
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
yes_rate: 0.7857142857142857
n_claims: 14
---

# scientific-array-io-performance-analysis

## Summary

Benchmarks and compares read performance of scientific array serialization formats — FITS versus chunked HDF5 (uncompressed and bitshuffle-compressed) — for full-file reads and partial slice reads of large multi-dimensional arrays, following the HDFITS Section 5.2 methodology.

## When to use

Use when you must quantify I/O performance of large multi-dim scientific arrays stored in FITS vs chunked/compressed HDF5, especially for random z-axis slice reads vs full-file reads.

## When NOT to use

- You only need full-file (contiguous, whole-volume) read or write throughput — the paper found full reads comparable (14.3/15.7/13.9 s) and the decisive advantage lies in partial access patterns
- You need a full characterization of arbitrary data access patterns on read performance — the article states 'full characterization of the effect of data access pattern upon read performance is beyond the scope of this paper'
- You are evaluating lossy compression algorithms — the article explicitly notes 'we have not compared lossy compression algorithms'

## Inputs

- 3D numpy array (e.g. (10000, 200, 200) int32) produced by an earlier generation task
- Serialized FITS file of that array
- Uncompressed chunked HDF5 file (chunk shape 1000x10x10) of that array
- Bitshuffle-compressed HDF5 file of that array

## Outputs

- io_benchmark_results.csv with per-format full-read and slice-read timing aggregates (median/mean over repetitions)
- Per-repetition timing records for 16 full-file reads and 16 slice-pair reads per format

## How to apply

Serialize the same array (e.g. a (10000, 200, 200) int32 volume) into three formats: a FITS file, an uncompressed chunked HDF5 file with a fixed chunk shape (e.g. 1000x10x10), and a bitshuffle-compressed HDF5 file (using fits2hdf/h5py). Time both full-file reads and random z-axis slice pair reads (e.g. 16 repetitions each), and defeat OS caching for every repetition by copying the file to a fresh cache-busting path before each timed read. Aggregate timings (median/mean) per format and write them to io_benchmark_results.csv. The rationale is that HDF5 chunking permits reading only the chunks covering a slice, whereas FITS must traverse the file sequentially, so slice reads should show orders-of-magnitude differences even when full-file reads are comparable (~14.3/15.7/13.9 s FITS/HDF5/bitshuffle vs ~21.1/0.24/0.21 s for slice reads in the paper).

## Related tools

- **fits2hdf** (Converts the array into FITS and HDFITS (HDF5) serializations used as benchmark subjects) — https://github.com/telegraphic/fits2hdf
- **h5py** (Python interface used to write chunked HDF5 datasets and time reads of full files and z-axis slices) — https://github.com/telegraphic/fits2hdf
- **bitshuffle** (HDF5 filter (filter number 32008) applied to the third serialization to test whether bit-level rearrangement compression affects read speed) — https://github.com/kiyo-masui/bitshuffle
- **Python** (Driver language for the benchmark loop, cache-busting copies, and aggregation of timings)

## Examples

```
from fits2hdf.io.fitsio import read_fits; from fits2hdf.io.hdfio import export_hdf; d = read_fits('data.fits'); export_hdf(d, 'data.h5')
```

## Evaluation signals

- Slice-pair read times for uncompressed HDF5 (~0.24 s) and bitshuffle HDF5 (~0.21 s) are two orders of magnitude faster than FITS (~21.1 s)
- Full-file read times across all three formats fall in the same broad range (~13.9–15.7 s) within generous tolerance for hardware variation
- Timing loop uses 16 repetitions per format per access pattern, each preceded by a fresh copy of the file to bust the OS cache
- io_benchmark_results.csv exists with one aggregate row per (format, access pattern) pair and finite, non-zero timings

## Limitations

- Reference timings (~14.3/15.7/13.9 s full reads; ~21.1/0.24/0.21 s slice reads) are hardware-dependent and should only be compared within generous tolerance
- The paper did not characterize arbitrary data access patterns; only full-file and random z-axis slice reads are covered
- No lossy compression algorithms were compared
- The HDF5 API provides only a low-level approach to loading data, which may complicate drop-in comparisons

## Evidence

- [readme] bitshuffle filter within HDF5/Numpy framework: [evidence span withheld — anchor status: external_doc]
