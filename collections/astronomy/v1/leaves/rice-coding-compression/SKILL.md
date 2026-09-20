---
name: rice-coding-compression
description: Use when benchmarking lossless compression ratios and compression times
  of image/array datasets across FITS Rice (FPACK) and HDF5 filters (LZF_SS, bitshuffle,
  GZIP) to decide between FITS and HDFITS storage.
when_to_use_negative:
- Evaluating lossy compression algorithms — the paper explicitly did not compare lossy
  compression
- Characterizing how data access patterns affect read performance — the authors state
  full characterization of access-pattern effects on read performance is beyond the
  paper's scope
- Datasets whose compression success depends on subtractive dithering — HDF5 currently
  has no equivalent to the FITS subtractive dithering technique
edam_operation: http://edamontology.org/operation_0282
tools:
- name: fits2hdf
  role: Conversion utility that applies the '-c' compression option to write HDFITS
    (LZF_SS, bitshuffle, GZIP) and converts FITS via CFITSIO; used to drive the benchmark
    compressions
  repo: https://github.com/telegraphic/fits2hdf
- name: bitshuffle
  role: HDF5 filter (number 32008) that bit-transposes typed binary data in ~8 kB
    blocks before LZF/LZ4 compression; the best-performing filter in the Table 1 benchmark
  repo: https://github.com/kiyo-masui/bitshuffle
- name: CFITSIO / FPACK
  role: Library providing the FITS Rice-coding image compression baseline (FPACK)
    against which HDF5 filters are compared
- name: hickle
  role: HDF5-based pickle replacement demonstrating LZF/GZIP compression of numpy
    arrays through h5py filter pipelines
  repo: https://github.com/telegraphic/hickle
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.05.001
    title: 'HDFITS: Porting the FITS data model to HDF5'
  source_content_hash: sha256:8cf55c20a7433a6076dd9ec582e8a2e6ca881c9f889a93eba9ed9590d5814c52
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/telegraphic/fits2hdf
  tools:
  - CFITSIO
  - CFITSIO / FPACK
  - Python
  - bitshuffle
  - fits2hdf
  - hickle
  edam_operation: http://edamontology.org/operation_0282
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# rice-coding-compression

## Summary

Benchmark lossless compression of uniformly distributed random-integer image arrays by comparing FITS Rice coding (FPACK/CFITSIO) against HDF5 filters (LZF_SS, bitshuffle, GZIP/deflate) via the fits2hdf '-c' compression option, producing a Table 1-style grid of compression ratios and times.

## When to use

Use when benchmarking lossless compression ratios and compression times of image/array datasets across FITS Rice (FPACK) and HDF5 filters (LZF_SS, bitshuffle, GZIP) to decide between FITS and HDFITS storage.

## When NOT to use

- Evaluating lossy compression algorithms — the paper explicitly did not compare lossy compression
- Characterizing how data access patterns affect read performance — the authors state full characterization of access-pattern effects on read performance is beyond the paper's scope
- Datasets whose compression success depends on subtractive dithering — HDF5 currently has no equivalent to the FITS subtractive dithering technique

## Inputs

- Uniformly distributed random-integer arrays at N=7, 15, 23, 31 (range 2^N), image sizes 2048x2048 and 8192x8192 (from task_001)
- FITS image files readable by CFITSIO/astropy
- HDF5 datasets writable with h5py filter pipelines (LZF_SS, bitshuffle, GZIP)

## Outputs

- Per-algorithm compressed file sizes and elapsed compression times
- Compression ratio (uncompressed size / compressed size) per algorithm x N x image-size cell
- compression_benchmark_table.csv (Table 1 grid)

## How to apply

Load the generated random-integer arrays spanning N=7,15,23,31 and image sizes 2048x2048 and 8192x8192. Compress each array with FPACK using Rice coding via the CFITSIO-based FITS image compression, recording compressed size and elapsed time. Compress the same arrays with the HDF5 filters LZF_SS, bitshuffle (HDF5 filter 32008, from the kiyo-masui package), and GZIP (deflate) using the fits2hdf '-c' compression option, again recording size and time. Compute the compression ratio as uncompressed size / compressed size for every algorithm x N x image-size combination, assemble the Table 1 grid (e.g. bitshuffle N=7, 2048x2048: ratio 3.84 in 0.04 s), and validate each cell against reference values within a small numerical tolerance. The rationale is that bitshuffle rearranges typed binary data at the bit level before LZF/LZ4 compression, exploiting bit-level uniformity that Rice coding only partially captures.

## Related tools

- **fits2hdf** (Conversion utility that applies the '-c' compression option to write HDFITS (LZF_SS, bitshuffle, GZIP) and converts FITS via CFITSIO; used to drive the benchmark compressions) — https://github.com/telegraphic/fits2hdf
- **bitshuffle** (HDF5 filter (number 32008) that bit-transposes typed binary data in ~8 kB blocks before LZF/LZ4 compression; the best-performing filter in the Table 1 benchmark) — https://github.com/kiyo-masui/bitshuffle
- **CFITSIO / FPACK** (Library providing the FITS Rice-coding image compression baseline (FPACK) against which HDF5 filters are compared)
- **hickle** (HDF5-based pickle replacement demonstrating LZF/GZIP compression of numpy arrays through h5py filter pipelines) — https://github.com/telegraphic/hickle

## Examples

```
fits2hdf input_dir output_dir -c
```

## Evaluation signals

- For 2048x2048 random integers with range 2^7, bitshuffle must reproduce ratio 3.84 in 0.04 s, FPACK 3.52 in 0.15 s, LZF_SS 3.55 in 0.12 s, GZIP 2.47 in 3.64 s (within small numerical tolerance)
- bitshuffle must consistently outperform FITS Rice and GZIP in both compression ratio and speed across the N x image-size grid
- Every (algorithm, N, image-size) cell of the Table 1 grid is populated and compression ratio equals uncompressed size / compressed size exactly
- Round-trip integrity: hdf2fits conversion of compressed HDFITS restores data identical to the source FITS arrays
- Compression ratio should be bounded below by ~1 for near-random data and approach 2^N/N-bit packing expectations for small N

## Limitations

- Bitshuffle relies on neighbouring dataset elements being correlated; only correlations spanning at least 24 elements can be exploited, and it alone does not compress (needs LZF/LZ4/ZSTD)
- Because bitshuffle modifies data at the bit level, entropy coders like GZIP and BZIP are unlikely to achieve significantly better compression than simpler duplicate-string elimination
- No lossy compression algorithms were compared, and HDF5 lacks an equivalent to FITS subtractive dithering
- Compression throughput is machine-dependent: bitshuffle gains come from SSE/AVX (roughly 1 byte per clock cycle, up to 2 on Haswell+), so timings may not transfer across hardware

## Evidence
