---
name: fits-image-compression-benchmarking
description: Use when comparing lossless compression performance (ratio and throughput)
  of FITS Rice/FPACK vs HDF5 filters (LZF_SS, bitshuffle, GZIP) on integer or astronomical
  image data.
when_to_use_negative:
- Data are floating-point astronomical values needing lossless recovery — the paper's
  Table 1 baseline is integer data only; for floats a FPACK-style subtractive-dithering
  pre-compression filter must be added first (HDF5 has 'currently no equivalent to
  the subtractive dithering').
- Lossy compression is the goal — the authors explicitly state 'we have not compared
  lossy compression algorithms'.
- Input is a MeasurementSet or already-compressed HDF5 dataset where compression benchmarking
  is moot (MS 'has no in-built compression capability' but bitshuffle-converted outputs
  already exist).
edam_operation: http://edamontology.org/operation_0282
tools:
- name: fits2hdf
  role: Converts FITS data into HDFITS HDF5 files; its '-c' compression option selects
    None/lzf/gzip/bitshuffle filters for the HDF5 side of the benchmark
  repo: https://github.com/telegraphic/fits2hdf
- name: CFITSIO/FPACK
  role: Provides the FITS Rice-compression baseline (fpack scaled-Rice mode) against
    which HDF5 filters are compared
- name: bitshuffle
  role: HDF5 dynamically loaded filter (number 32008) that bit-transposes typed binary
    data before LZF/LZ4 compression; the best-performing filter in the benchmark
  repo: https://github.com/kiyo-masui/bitshuffle
- name: h5py
  role: Pythonic HDF5 interface used to apply filter pipelines and read compressed
    datasets
- name: hickle
  role: HDF5-based serializer exposing h5py compression keywords (lzf, gzip, shuffle,
    fletcher32) for HDF5-side compression experiments
  repo: https://github.com/telegraphic/hickle
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1016/j.ascom.2015.05.001
    title: 'HDFITS: Porting the FITS data model to HDF5'
  source_content_hash: sha256:506c6a24ae56bbcd9e7c69b232e8b5ddd28952b98954e242368caf411a86d010
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/telegraphic/fits2hdf
  tools:
  - CFITSIO
  - CFITSIO/FPACK
  - Python
  - bitshuffle
  - fits2hdf
  - h5py
  - hickle
  edam_operation: http://edamontology.org/operation_0282
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# fits-image-compression-benchmarking

## Summary

Benchmark lossless compression ratios and compression times of FITS Rice (FPACK) against HDF5 filters (LZF_SS, bitshuffle, GZIP) using fits2hdf, reproducing the HDFITS Table 1 comparison on generated random-integer images.

## When to use

Use when comparing lossless compression performance (ratio and throughput) of FITS Rice/FPACK vs HDF5 filters (LZF_SS, bitshuffle, GZIP) on integer or astronomical image data.

## When NOT to use

- Data are floating-point astronomical values needing lossless recovery — the paper's Table 1 baseline is integer data only; for floats a FPACK-style subtractive-dithering pre-compression filter must be added first (HDF5 has 'currently no equivalent to the subtractive dithering').
- Lossy compression is the goal — the authors explicitly state 'we have not compared lossy compression algorithms'.
- Input is a MeasurementSet or already-compressed HDF5 dataset where compression benchmarking is moot (MS 'has no in-built compression capability' but bitshuffle-converted outputs already exist).

## Inputs

- Uniformly distributed random-integer arrays (task_001 outputs) spanning N=7,15,23,31 and image sizes 2048x2048 and 8192x8192
- FITS images readable via fits2hdf.io.fitsio.read_fits into the IDI in-memory interchange format
- CFITSIO/FPACK toolchain for Rice-coded FITS compression

## Outputs

- compression_benchmark_table.csv with compression ratio and compression time for every algorithm x N x image-size combination
- Per-cell validation against reported reference values (e.g. bitshuffle N=7, 2048x2048: ratio 3.84, time 0.04 s)

## How to apply

Generate or load uniformly distributed random-integer arrays across the N and image-size grid (N=7,15,23,31; sizes up to 2048x2048 and 8192x8192) from task_001. Compress each array with FPACK's Rice coding via CFITSIO to produce the FITS baseline, then compress with the HDF5 filters LZF_SS, bitshuffle (from kiyo-masui/bitshuffle), and GZIP (deflate) using the fits2hdf '-c' compression option, recording compressed size and elapsed wall-clock time for every algorithm x N x image-size cell. Compute the compression ratio as uncompressed size / compressed size and assemble results into the Table 1 grid (compression_benchmark_table.csv). Validate each cell against reference values within a small numerical tolerance — e.g. bitshuffle at N=7, 2048x2048 should give ratio 3.84 in 0.04 s, beating FITS Rice (3.52, 0.15 s), LZF_SS (3.55, 0.12 s), and GZIP (2.47, 3.64 s). Rationale: bitshuffle's bit-level transposition (HDF5 filter 32008) exploits bit correlations that GZIP cannot, so it is expected to dominate both ratio and speed on typed binary data.

## Related tools

- **fits2hdf** (Converts FITS data into HDFITS HDF5 files; its '-c' compression option selects None/lzf/gzip/bitshuffle filters for the HDF5 side of the benchmark) — https://github.com/telegraphic/fits2hdf
- **CFITSIO/FPACK** (Provides the FITS Rice-compression baseline (fpack scaled-Rice mode) against which HDF5 filters are compared)
- **bitshuffle** (HDF5 dynamically loaded filter (number 32008) that bit-transposes typed binary data before LZF/LZ4 compression; the best-performing filter in the benchmark) — https://github.com/kiyo-masui/bitshuffle
- **h5py** (Pythonic HDF5 interface used to apply filter pipelines and read compressed datasets)
- **hickle** (HDF5-based serializer exposing h5py compression keywords (lzf, gzip, shuffle, fletcher32) for HDF5-side compression experiments) — https://github.com/telegraphic/hickle

## Examples

```
fits2hdf input_dir output_dir -c bitshuffle
```

## Evaluation signals

- Reproduced reference cells: bitshuffle N=7, 2048x2048 gives ratio 3.84 in 0.04 s; FITS Rice 3.52/0.15 s; LZF_SS 3.55/0.12 s; GZIP 2.47/3.64 s (within small numerical tolerance).
- bitshuffle consistently outperforms FITS Rice and GZIP in both compression ratio and speed across the N x image-size grid.
- Round-trip decompression (HDF5 to FITS and back via hdf2fits) recovers the original pixels exactly for all lossless filters.
- compression_benchmark_table.csv contains a complete algorithm x N x image-size grid with no missing cells and monotone ratio behavior w.r.t. bit-depth N.
- Compressed FITS files remain readable by legacy FITS software, confirming the backwards-compatibility requirement.

## Limitations

- The paper's compression comparison is on integer data only; HDF5 currently has no equivalent to FITS subtractive dithering for floats.
- Lossy compression algorithms were not compared.
- Bitshuffle's algorithm relies on neighbouring elements being highly correlated; on truly random data (e.g. N=31) compression gains shrink, and it exploits correlations spanning at least 24 elements.
- HDF5 lacks widespread adoption in astronomy because it is not well supported by data reduction software packages and image viewers.

## Evidence

- [readme] bitshuffle README algorithm: [evidence span withheld — anchor status: external_doc]
