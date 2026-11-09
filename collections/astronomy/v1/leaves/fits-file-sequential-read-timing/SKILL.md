---
name: fits-file-sequential-read-timing
description: Use when comparing I/O performance of serialised scientific file formats
  (FITS vs HDF5/HDFITS) for large 3D datasets, especially when random slice access
  speed, not just full-file throughput, is the deciding factor.
when_to_use_negative:
- You only need to convert files between FITS and HDF5 formats rather than measure
  read performance — use the plain fits2hdf/hdf2fits conversion workflow instead.
- A full characterization of arbitrary data-access patterns on read performance is
  required — the paper states this is beyond its scope and only two access patterns
  (full-file and random z-slice) are benchmarked.
tools:
- name: fits2hdf
  role: Converts the 3D array to FITS and HDFITS/HDF5 representations whose read speeds
    are then benchmarked
  repo: https://github.com/telegraphic/fits2hdf
- name: h5py
  role: Python interface used to write chunked datasets (chunk shape 1000x10x10) and
    time dataset reads
- name: bitshuffle
  role: HDF5 filter (ID 32008) providing bitshuffle-compressed storage variant of
    the HDF5 file under test
  repo: https://github.com/kiyo-masui/bitshuffle
- name: HDF5
  role: Underlying chunked binary format whose slice-read performance is contrasted
    with FITS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.05.001
    title: 'HDFITS: Porting the FITS data model to HDF5'
  source_content_hash: sha256:92577f8f971d28340571dd8d63a6f160a6d8403db7120ed5997b345afc8a0a19
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# fits-file-sequential-read-timing

## Summary

Benchmark methodology for timing FITS versus chunked HDF5 (uncompressed and bitshuffle-compressed) reads of large 3D arrays, comparing full-file sequential reads against random z-axis slice reads to expose FITS's lack of chunked access.

## When to use

Use when comparing I/O performance of serialised scientific file formats (FITS vs HDF5/HDFITS) for large 3D datasets, especially when random slice access speed, not just full-file throughput, is the deciding factor.

## When NOT to use

- You only need to convert files between FITS and HDF5 formats rather than measure read performance — use the plain fits2hdf/hdf2fits conversion workflow instead.
- A full characterization of arbitrary data-access patterns on read performance is required — the paper states this is beyond its scope and only two access patterns (full-file and random z-slice) are benchmarked.

## Inputs

- 3D numpy array, e.g. (10000, 200, 200) int32 image/data cube
- Serialized copies of the same array as FITS, uncompressed chunked HDF5, and bitshuffle-compressed HDF5 files

## Outputs

- Aggregated per-format read timings (full-file read and z-axis slice read) in a CSV such as io_benchmark_results.csv
- Comparison against reference timings (~14.3/15.7/13.9 s full reads; ~21.1/0.24/0.21 s slice reads)

## How to apply

Serialize the same 3D array (e.g. a (10000, 200, 200) int32 radio-astronomy image cube) into three files: a FITS file, an uncompressed chunked HDF5 file (chunk shape 1000x10x10), and a bitshuffle-compressed HDF5 file using fits2hdf/h5py. Time both a full-file read and a read of a pair of randomly chosen z-axis slices, repeated 16 times per format, forcing each repetition through a fresh cache-busting copy of the file so OS page-cache effects do not skew results. Aggregate per-format timings (median/mean) and record them, e.g. to io_benchmark_results.csv. The expected outcome, per the paper's Section 5.2 reference values, is near-parity for full reads (~14.3/15.7/13.9 s for FITS / chunked HDF5 / bitshuffle HDF5) but orders-of-magnitude faster slice reads for chunked HDF5 (~0.24/0.21 s) versus FITS (~21.1 s), consistent with the paper's claim of HDF5 reading 'up to 100x of FITS in some use cases'.

## Related tools

- **fits2hdf** (Converts the 3D array to FITS and HDFITS/HDF5 representations whose read speeds are then benchmarked) — https://github.com/telegraphic/fits2hdf
- **h5py** (Python interface used to write chunked datasets (chunk shape 1000x10x10) and time dataset reads)
- **bitshuffle** (HDF5 filter (ID 32008) providing bitshuffle-compressed storage variant of the HDF5 file under test) — https://github.com/kiyo-masui/bitshuffle
- **HDF5** (Underlying chunked binary format whose slice-read performance is contrasted with FITS)

## Examples

```
fits2hdf input_dir output_dir <options>  # then time reads: import h5py; dset = h5py.File('cube_bshuf.h5','r')['data']; z = dset[(z1, slice(None), slice(None))]
```

## Evaluation signals

- Full-file read timings for all three formats fall within generous tolerance of ~13.9–15.7 s, confirming full-read parity.
- Chunked HDF5 slice reads (~0.24 s uncompressed, ~0.21 s bitshuffle) are two orders of magnitude faster than FITS slice reads (~21.1 s).
- Every timed repetition reads a fresh cache-busting copy of the file, so no repetition is inflated by OS cache hits.
- Aggregated statistics (median/mean over 16 repetitions) for both access patterns and all three formats are present in io_benchmark_results.csv.
- Re-reading data from the HDF5 files returns the original (10000, 200, 200) int32 array bit-for-bit.

## Limitations

- Read performance depends heavily on hardware; measured timings must be compared to the paper's reference values only within generous tolerance.
- Only two data-access patterns (full sequential read and random z-axis slice pair) are benchmarked; full characterization of access-pattern effects on read performance is explicitly out of scope.
- Only bitshuffle (lossless) compression is compared; the paper did not compare lossy compression algorithms.

## Evidence

- [readme] bitshuffle as an HDF5 filter for compression benchmarking: [evidence span withheld — anchor status: external_doc]
