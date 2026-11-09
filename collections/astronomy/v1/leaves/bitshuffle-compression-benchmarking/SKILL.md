---
name: bitshuffle-compression-benchmarking
description: Use when you need to quantify I/O speedups from switching large 3D (e.g.
  radio-astronomy) integer arrays from FITS to chunked or bitshuffle-compressed HDF5,
  especially for partial (slice) reads of a large cube.
when_to_use_negative:
- Full characterization of data access patterns on read performance is beyond the
  paper's scope — do not generalize the benchmark beyond full-file and random z-axis
  slice reads
- The paper did not compare lossy compression algorithms, so this benchmark cannot
  rank lossy codecs
- Input is small or access is always full-file sequential, where FITS and HDF5 full-read
  times are comparable (~13.9–15.7 s) and the HDF5 advantage disappears
tools:
- name: fits2hdf
  role: Converts the FITS-serialized cube into the HDFITS HDF5 representation used
    for the HDF5 benchmark arms
  repo: https://github.com/telegraphic/fits2hdf
- name: bitshuffle
  role: HDF5 filter (number 32008) that rearranges typed binary data at the bit level
    for improved compression of radio-astronomy datasets
  repo: https://github.com/kiyo-masui/bitshuffle
- name: h5py
  role: Pythonic HDF5 interface used to create chunked/compressed datasets and to
    time slice reads
  repo: https://github.com/telegraphic/fits2hdf
- name: Python
  role: 'Benchmark driver: loads the numpy cube, performs cache-busted timed reads,
    and writes io_benchmark_results.csv'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.05.001
    title: 'HDFITS: Porting the FITS data model to HDF5'
  source_content_hash: sha256:c932957034aec13a1bd66e754434849a837859cfc9484574077f9f59dc21a37c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/telegraphic/fits2hdf
  tools:
  - Python
  - bitshuffle
  - fits2hdf
  - h5py
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# bitshuffle-compression-benchmarking

## Summary

Benchmark read performance of the bitshuffle-compressed HDF5 (HDFITS) representation of astronomical data against uncompressed chunked HDF5 and legacy FITS, for both full-file reads and random z-axis slice reads. Grounded in the Section 5.2 read-speed experiments of Price et al. (2015), where bitshuffle+HDF5 reads slices orders of magnitude faster than FITS.

## When to use

Use when you need to quantify I/O speedups from switching large 3D (e.g. radio-astronomy) integer arrays from FITS to chunked or bitshuffle-compressed HDF5, especially for partial (slice) reads of a large cube.

## When NOT to use

- Full characterization of data access patterns on read performance is beyond the paper's scope — do not generalize the benchmark beyond full-file and random z-axis slice reads
- The paper did not compare lossy compression algorithms, so this benchmark cannot rank lossy codecs
- Input is small or access is always full-file sequential, where FITS and HDF5 full-read times are comparable (~13.9–15.7 s) and the HDF5 advantage disappears

## Inputs

- 3D int32 numpy array (e.g. shape (10000, 200, 200)) produced by task_001
- Serialized FITS file of the same cube
- Uncompressed chunked HDF5 file (chunk shape 1000x10x10)
- Bitshuffle-compressed HDF5 file (bitshuffle filter 32008, e.g. with LZ4)

## Outputs

- io_benchmark_results.csv with per-format full-read and slice-read timings aggregated over 16 repetitions
- Comparison of measured timings against reference values (~14.3/15.7/13.9 s full reads; ~21.1/0.24/0.21 s slice reads)

## How to apply

Serialize the same 3D int32 array (here a (10000, 200, 200) cube) three ways with fits2hdf/h5py: as a FITS file, as an uncompressed chunked HDF5 dataset (chunk shape 1000x10x10 in the paper), and as a bitshuffle-compressed HDF5 dataset (bitshuffle is HDF5 filter 32008, well matched to LZ4/LZF for radio-astronomy data). For each format, time 16 repetitions of a full-file read and 16 repetitions of reading a pair of randomly chosen z-axis slices, forcing each repetition through a fresh cache-busting copy of the file to defeat operating-system page-cache effects. Aggregate per-format timings (median/mean over repetitions) and compare: the paper reports full reads of ~14.3/15.7/13.9 s for FITS/uncompressed-HDF5/bitshuffle-HDF5 (comparable) but slice reads of ~21.1 s for FITS versus ~0.24 s and ~0.21 s for HDF5, i.e. roughly a 100x slice-read advantage that justifies HDF5 chunking for partial access.

## Related tools

- **fits2hdf** (Converts the FITS-serialized cube into the HDFITS HDF5 representation used for the HDF5 benchmark arms) — https://github.com/telegraphic/fits2hdf
- **bitshuffle** (HDF5 filter (number 32008) that rearranges typed binary data at the bit level for improved compression of radio-astronomy datasets) — https://github.com/kiyo-masui/bitshuffle
- **h5py** (Pythonic HDF5 interface used to create chunked/compressed datasets and to time slice reads) — https://github.com/telegraphic/fits2hdf
- **Python** (Benchmark driver: loads the numpy cube, performs cache-busted timed reads, and writes io_benchmark_results.csv)

## Examples

```
python benchmark_io.py --input cube_10000x200x200.npy --formats fits,hdf5_chunked,hdf5_bitshuffle --reps 16 --out io_benchmark_results.csv
```

## Evaluation signals

- HDF5 (uncompressed and bitshuffle) slice reads must be orders of magnitude faster than FITS slice reads (~0.24/0.21 s vs ~21.1 s), confirming the paper's up-to-100x claim
- Full-file read times should be roughly comparable across all three formats (~13.9–15.7 s), within generous tolerance for hardware variation
- Each timing repetition must run against a fresh cache-busting copy of the file so OS page cache does not contaminate measurements
- io_benchmark_results.csv exists with 3 formats × 2 read patterns × 16 repetitions of raw timings plus aggregates
- Data read back from all three serializations must be bit-identical to the source (10000, 200, 200) int32 array

## Limitations

- Full characterization of the effect of data access pattern upon read performance is beyond the scope of the paper
- No comparison of lossy compression algorithms was performed
- Absolute timings vary with hardware; the paper's values (e.g. 21.1 s FITS vs 0.21 s bitshuffle-HDF5 slice reads) must be compared within generous tolerance
- HDF5 API provides only a low-level approach to loading data, which can complicate transparent usage versus astropy.io.fits

## Evidence

- [abstract] HDFITS allows faster reading of data, up to 100x of FITS in some use cases, and improved compression: "We show that HDFITS allows faster reading of data (up to 100x of FITS in some use cases), and improved compression"
- [methods] bitshuffle compression for radio astronomy data: "`bitshuffle <https://github.com/kiyo-masui/bitshuffle>`_ compression (good for radio astronomy data)"
- [discussion] LEDA converts raw interferometric data to bitshuffle compressed HDF5 files: "to convert raw in-
terferometric data to bitshuffle compressed HDF5 files."
- [readme] Bitshuffle is an HDF5 filter for improving compression of typed binary data: [evidence span withheld — anchor status: external_doc]
- [readme] Bitshuffle filter number 32008: [evidence span withheld — anchor status: external_doc]
