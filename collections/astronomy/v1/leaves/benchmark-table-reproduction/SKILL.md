---
name: benchmark-table-reproduction
description: Use when reproducing a published compression benchmark table that compares
  specific lossless codecs (Rice/FITS, LZF, bitshuffle, GZIP) on synthetic integer
  arrays and requires cell-by-cell numerical validation.
when_to_use_negative:
- The task requires comparing lossy compression algorithms — the paper explicitly
  did not compare lossy compression
- The data are real astronomical images with correlated or dithered values rather
  than uniform random integers — bitshuffle relies on neighbouring elements being
  highly correlated and HDF5 has no equivalent to FITS subtractive dithering
- The goal is read-speed benchmarking of HDFITS vs FITS rather than compression ratio/time
  reproduction
tools:
- name: fits2hdf
  role: Conversion utility applying the HDF5 filters (LZF_SS, bitshuffle, GZIP) via
    the '-c' compression option; used with Python, HDF5, and CFITSIO to build the
    benchmark
  repo: https://github.com/telegraphic/fits2hdf
- name: bitshuffle
  role: HDF5 filter (filter number 32008) that bit-level transposes typed binary data
    before compression; the top-performing codec in the benchmark
  repo: https://github.com/kiyo-masui/bitshuffle
- name: CFITSIO
  role: Library underlying FPACK Rice coding used for the FITS compression baseline
- name: hickle
  role: HDF5-based serializer (from the same group) demonstrating LZF/GZIP compression
    options in h5py; supplementary context for the HDF5 filter workflow
  repo: https://github.com/telegraphic/hickle
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.05.001
    title: 'HDFITS: Porting the FITS data model to HDF5'
  source_content_hash: sha256:943d8c6a858cbcf98adc0300592fe910268c1692826378f69c7a6daabd2f1a9e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/telegraphic/fits2hdf
  tools:
  - CFITSIO
  - Python
  - bitshuffle
  - fits2hdf
  - hickle
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# benchmark-table-reproduction

## Summary

Reproduce the paper's Table 1 lossless compression benchmark by compressing generated uniformly distributed random-integer images (N=7,15,23,31; sizes 2048x2048 and 8192x8192) with FITS Rice (FPACK), HDF5 LZF_SS, HDF5 bitshuffle, and GZIP, then validating every ratio/time cell against the reported reference values.

## When to use

Use when reproducing a published compression benchmark table that compares specific lossless codecs (Rice/FITS, LZF, bitshuffle, GZIP) on synthetic integer arrays and requires cell-by-cell numerical validation.

## When NOT to use

- The task requires comparing lossy compression algorithms — the paper explicitly did not compare lossy compression
- The data are real astronomical images with correlated or dithered values rather than uniform random integers — bitshuffle relies on neighbouring elements being highly correlated and HDF5 has no equivalent to FITS subtractive dithering
- The goal is read-speed benchmarking of HDFITS vs FITS rather than compression ratio/time reproduction

## Inputs

- Random-integer arrays from task_001 spanning N=7,15,23,31 and image sizes 2048x2048 and 8192x8192 (uniformly distributed values)
- fits2hdf toolchain with the '-c' compression option (LZF_SS, bitshuffle, GZIP)
- CFITSIO-based FPACK for FITS Rice compression

## Outputs

- compression_benchmark_table.csv with one row per algorithm x N x image-size combination, containing compression ratio and compression time
- Validated Table 1 grid with each cell checked against reference values within tolerance

## How to apply

Generate or load the uniformly distributed random-integer arrays spanning the N (7, 15, 23, 31) by image-size (2048x2048, 8192x8192) grid from task_001. Compress each array with FPACK using Rice coding via CFITSIO, and with the HDF5 filters LZF_SS, bitshuffle (from the kiyo-masui/bitshuffle package), and GZIP (deflate) using the fits2hdf '-c' compression option, recording compressed size and elapsed time for every algorithm x N x size combination. Compute the compression ratio as uncompressed size / compressed size and assemble the grid into compression_benchmark_table.csv. Validate each cell against reference values — e.g. bitshuffle at N=7 on a 2048x2048 image should yield ratio 3.84 in 0.04 s, versus FITS Rice (3.52, 0.15 s), LZF_SS (3.55, 0.12 s), and GZIP (2.47, 3.64 s) — within a small numerical tolerance. The rationale is that bitshuffle rearranges typed binary data at the bit level before LZF/LZ4-style duplicate-string elimination, which is why it consistently beats Rice and GZIP on ratio and speed for this data.

## Related tools

- **fits2hdf** (Conversion utility applying the HDF5 filters (LZF_SS, bitshuffle, GZIP) via the '-c' compression option; used with Python, HDF5, and CFITSIO to build the benchmark) — https://github.com/telegraphic/fits2hdf
- **bitshuffle** (HDF5 filter (filter number 32008) that bit-level transposes typed binary data before compression; the top-performing codec in the benchmark) — https://github.com/kiyo-masui/bitshuffle
- **CFITSIO** (Library underlying FPACK Rice coding used for the FITS compression baseline)
- **hickle** (HDF5-based serializer (from the same group) demonstrating LZF/GZIP compression options in h5py; supplementary context for the HDF5 filter workflow) — https://github.com/telegraphic/hickle

## Examples

```
git clone https://github.com/telegraphic/fits2hdf && fits2hdf input_dir output_dir -c bitshuffle
```

## Evaluation signals

- Cell-level validation: bitshuffle at N=7, 2048x2048 must reproduce ratio 3.84 and time 0.04 s within a small numerical tolerance
- Relative ordering check: bitshuffle consistently outperforms FITS Rice and GZIP in both compression ratio and speed across the N x size grid
- Completeness check: compression_benchmark_table.csv contains every algorithm x N x image-size combination (4 algorithms x 4 N values x 2 sizes)
- Invariant check: reported compression ratio equals uncompressed size / compressed size for each cell
- Sanity check: GZIP is the slowest codec (e.g. 3.64 s at N=7, 2048x2048) while bitshuffle and LZF_SS are sub-0.2 s

## Limitations

- No comparison of lossy compression algorithms was performed
- Results are specific to uniformly distributed random-integer data; bitshuffle's algorithm relies on neighbouring elements of a dataset being highly correlated, and correlations must span at least 24 elements to be exploited
- HDF5 currently has no equivalent to the FITS subtractive dithering technique, which can affect compression of real astronomical data
- Absolute timings are machine-dependent; only relative comparisons (ratio and speed ordering) should be trusted across systems

## Evidence

- [results] bitshuffle achieved a compression ratio of 3.84 in 0.04 s, outperforming FITS Rice (ratio 3.52, 0.15 s), HDF5 LZF_SS (3.55, 0.12 s), and GZIP (2.47, 3.64 s): [evidence span withheld — anchor status: not_found]
- [readme] README: bitshuffle algorithm description: [evidence span withheld — anchor status: external_doc]
- [methods] Article methods: bitshuffle repo: "`bitshuffle <https://github.com/kiyo-masui/bitshuffle>`_ compression (good for radio astronomy data)"
- [abstract] Article abstract: HDFITS compression claim: "We show that HDFITS allows faster reading of data (up to 100x of FITS in some use cases), and improved compression (higher compression ratios and higher throughput)"
- [discussion] Article discussion: no lossy comparison: [evidence span withheld — anchor status: not_found]
