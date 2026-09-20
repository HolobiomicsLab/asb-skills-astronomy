---
name: hdf5-filter-compression-evaluation
description: Use when a user needs to benchmark lossless compression filters — FITS
  Rice, HDF5 LZF_SS, bitshuffle, GZIP — on synthetic random-integer image grids and
  reproduce a ratio-vs-time comparison table.
when_to_use_negative:
- The data are real astronomical (correlated) images rather than synthetic random
  integers — bitshuffle's gains depend on within-block structure and the paper's Table
  1 values will not transfer
- A comparison of lossy compression algorithms is required — the article explicitly
  states 'we have not compared lossy compression algorithms'
- The goal is read-throughput benchmarking or data-model conversion (FITS <-> HDFITS)
  rather than compression ratio/time evaluation
tools:
- name: fits2hdf
  role: Conversion utility used with the '-c' compression option to apply HDF5 filters
    (LZF_SS, bitshuffle, GZIP) to the arrays
  repo: https://github.com/telegraphic/fits2hdf
- name: bitshuffle
  role: HDF5 dynamically loaded filter (filter ID 32008) that bit-transposes typed
    binary data before LZF/LZ4 compression
  repo: https://github.com/kiyo-masui/bitshuffle
- name: CFITSIO
  role: Library behind FPACK, providing FITS Rice-coding compression for the FITS
    baseline in the benchmark
- name: Python
  role: Scripting language used to generate arrays, drive compression, compute ratios,
    and assemble the Table 1 CSV
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.05.001
    title: 'HDFITS: Porting the FITS data model to HDF5'
  source_content_hash: sha256:5aa65e5fd67e92e43702b64839c34b298e8de2099833decd6a443159f87ff73e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/telegraphic/fits2hdf
  tools:
  - CFITSIO
  - HDF5
  - Python
  - bitshuffle
  - fits2hdf
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# hdf5-filter-compression-evaluation

## Summary

A benchmarking method for comparing lossless compression ratios and compression times of FITS Rice (FPACK via CFITSIO) against HDF5 filters (LZF_SS, bitshuffle, GZIP/deflate) on uniformly distributed random-integer images, as done in the HDFITS paper's Table 1.

## When to use

Use when a user needs to benchmark lossless compression filters — FITS Rice, HDF5 LZF_SS, bitshuffle, GZIP — on synthetic random-integer image grids and reproduce a ratio-vs-time comparison table.

## When NOT to use

- The data are real astronomical (correlated) images rather than synthetic random integers — bitshuffle's gains depend on within-block structure and the paper's Table 1 values will not transfer
- A comparison of lossy compression algorithms is required — the article explicitly states 'we have not compared lossy compression algorithms'
- The goal is read-throughput benchmarking or data-model conversion (FITS <-> HDFITS) rather than compression ratio/time evaluation

## Inputs

- Generated uniformly distributed random-integer NumPy/HDF5 arrays spanning N in {2^7, 2^15, 2^23, 2^31} at image sizes 2048x2048 and 8192x8192
- fits2hdf-compatible FITS/HDF5 files of those arrays for applying the '-c' compression option
- CFITSIO/FPACK-capable FITS files for Rice compression

## Outputs

- compression_benchmark_table.csv with rows for every algorithm x N x image-size combination
- Per-cell compression ratio (uncompressed size / compressed size) and compression time in seconds
- Validated Table 1 grid matching paper reference values within tolerance

## How to apply

Generate (or load) random-integer arrays spanning the N (bit-range 2^7, 2^15, 2^23, 2^31) and image-size grid (2048x2048, 8192x8192) from the preceding data-generation task. Compress each array with FPACK Rice coding (CFITSIO-based FITS image compression), then with the HDF5 filters LZF_SS, bitshuffle (from kiyo-masui/bitshuffle), and GZIP via the fits2hdf '-c' compression option, recording compressed size and elapsed time for every algorithm x N x image-size combination. Compute the compression ratio as uncompressed size / compressed size and assemble the grid into a Table 1 CSV. Validate each cell against reference values (e.g. bitshuffle N=7, 2048x2048: ratio 3.84, time 0.04 s vs FITS Rice 3.52/0.15 s, LZF_SS 3.55/0.12 s, GZIP 2.47/3.64 s) within a small numerical tolerance. Rationale: bitshuffle operates at the bit level (like HDF5's shuffle filter, but transposing bits within ~8 kB blocks) and is matched to simple duplicate-string-elimination codecs such as LZF/LZ4, which is why it beats byte-oriented GZIP on random integers.

## Related tools

- **fits2hdf** (Conversion utility used with the '-c' compression option to apply HDF5 filters (LZF_SS, bitshuffle, GZIP) to the arrays) — https://github.com/telegraphic/fits2hdf
- **bitshuffle** (HDF5 dynamically loaded filter (filter ID 32008) that bit-transposes typed binary data before LZF/LZ4 compression) — https://github.com/kiyo-masui/bitshuffle
- **CFITSIO** (Library behind FPACK, providing FITS Rice-coding compression for the FITS baseline in the benchmark)
- **Python** (Scripting language used to generate arrays, drive compression, compute ratios, and assemble the Table 1 CSV)

## Examples

```
fits2hdf input_dir output_dir -c bitshuffle
```

## Evaluation signals

- compression_benchmark_table.csv contains an entry for every algorithm (FPACK Rice, LZF_SS, bitshuffle, GZIP) x N (7, 15, 23, 31) x image size (2048x2048, 8192x8192) combination
- Each cell's ratio and time matches the paper's reference values within small tolerance (e.g. bitshuffle N=7, 2048x2048: ratio 3.84, time 0.04 s)
- Compression ratio is consistently computed as uncompressed size / compressed size
- bitshuffle outperforms FITS Rice and GZIP in both ratio and speed on the random-integer datasets, and GZIP is clearly slowest (2.47 ratio, 3.64 s at N=7, 2048x2048)
- Decompression of each compressed dataset reproduces the original array exactly (lossless round-trip invariant)

## Limitations

- Only lossless algorithms were benchmarked; the article states 'we have not compared lossy compression algorithms'
- HDF5 has no equivalent to FITS subtractive dithering, which can limit compression gains on certain data
- Bitshuffle exploits correlations spanning at least ~24 neighbouring dataset elements; random data beyond the block-internal bit-level structure yields limited benefit and results depend on hardware SSE/AVX support
- GZIP and similar entropy coders are unlikely to beat simple duplicate-string-elimination codecs (LZF, LZ4) when applied after bit-level shuffling

## Evidence

- [results] bitshuffle achieved a compression ratio of 3.84 in 0.04 s, outperforming FITS Rice (ratio 3.52, 0.15 s), HDF5 LZF_SS (3.55, 0.12 s), and GZIP (2.47, 3.64 s): [evidence span withheld — anchor status: not_found]
- [readme] Bitshuffle is an algorithm that rearranges typed, binary data for improving compression: [evidence span withheld — anchor status: external_doc]
- [readme] Bitshuffle is HDF5 filter number ``32008``.: [evidence span withheld — anchor status: external_doc]
