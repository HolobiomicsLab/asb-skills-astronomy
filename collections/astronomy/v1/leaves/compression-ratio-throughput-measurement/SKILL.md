---
name: compression-ratio-throughput-measurement
description: Decision support for choosing a lossless compression filter when storing
  large typed binary arrays (e.g. FITS images or HDF5 datasets), where both compression
  ratio and compression throughput matter.
when_to_use_negative:
- Evaluating lossy compression algorithms — the paper explicitly states 'we have not
  compared lossy compression algorithms'
- Data where neighbouring elements lack correlation — bitshuffle 'relies on neighbouring
  elements of a dataset being highly correlated', so random float or truly incompressible
  data will show little gain
- When throughput on entropy-coded formats is acceptable — GZIP/BZIP-style sophisticated
  entropy reduction is unlikely to beat simple shuffle-plus-LZF/LZ4 pipelines on bit-shuffled
  typed binary data anyway
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: fits2hdf
  role: CLI utility used with the '-c' compression option to write arrays through
    HDF5 filters (LZF_SS, bitshuffle, GZIP) and to convert to/from FITS
  repo: https://github.com/telegraphic/fits2hdf
- name: bitshuffle
  role: HDF5 filter (number 32008) that bit-transposes typed binary data before LZF/LZ4
    compression; the fastest, highest-ratio codec in the benchmark
  repo: https://github.com/kiyo-masui/bitshuffle
- name: CFITSIO
  role: Library underlying FPACK, used to apply FITS Rice compression for the FITS
    baseline
- name: hickle
  role: HDF5-based dump utility that exposes h5py compression keywords (lzf, gzip,
    shuffle, chunks) usable for compressed storage comparisons
  repo: https://github.com/telegraphic/hickle
- name: Python
  role: Driver language for generating arrays, timing compression, computing ratios,
    and writing the CSV table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.05.001
    title: 'HDFITS: Porting the FITS data model to HDF5'
  source_content_hash: sha256:5d0ff65948de1a5c3305a875581ebc6e391d752880eacbc800c8693b98952f3c
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# compression-ratio-throughput-measurement

## Summary

Benchmark lossless compression filters (FITS Rice/FPACK, HDF5 LZF_SS, bitshuffle, GZIP) on identical datasets, recording compression ratio (uncompressed/compressed size) and elapsed compression time per algorithm, as in the HDFITS Table 1 comparison.

## When to use

Decision support for choosing a lossless compression filter when storing large typed binary arrays (e.g. FITS images or HDF5 datasets), where both compression ratio and compression throughput matter.

## When NOT to use

- Evaluating lossy compression algorithms — the paper explicitly states 'we have not compared lossy compression algorithms'
- Data where neighbouring elements lack correlation — bitshuffle 'relies on neighbouring elements of a dataset being highly correlated', so random float or truly incompressible data will show little gain
- When throughput on entropy-coded formats is acceptable — GZIP/BZIP-style sophisticated entropy reduction is unlikely to beat simple shuffle-plus-LZF/LZ4 pipelines on bit-shuffled typed binary data anyway

## Inputs

- Random-integer NumPy arrays spanning N=7,15,23,31 bit ranges at 2048x2048 and 8192x8192 pixels (from task_001)
- Uncompressed baseline file sizes for each array
- fits2hdf-supported FITS/HDF5 dataset containers

## Outputs

- compression_benchmark_table.csv with ratio and time per algorithm x N x image-size cell
- Per-cell (compression ratio, elapsed time) pairs validated against Table 1 reference values
- Ranking of filters by ratio and throughput

## How to apply

Generate or load the same arrays for every codec (e.g. uniformly distributed random integers across a range N=2^7 to 2^31 at image sizes 2048x2048 and 8192x8192). Compress each array with FPACK via CFITSIO (Rice coding) and with HDF5 filters LZF_SS, bitshuffle (kiyo-masui/bitshuffle, HDF5 filter 32008), and GZIP (deflate) via fits2hdf's '-c' compression option, recording compressed size and wall-clock time for every algorithm x N x size cell. Compute compression ratio as uncompressed size divided by compressed size. Assemble the grid (e.g. compression_benchmark_table.csv) and validate cells against reference values such as bitshuffle N=7, 2048x2048: ratio 3.84 in 0.04 s, versus FITS Rice (3.52, 0.15 s), LZF_SS (3.55, 0.12 s), and GZIP (2.47, 3.64 s). Bitshuffle is expected to win on both axes because, unlike byte-level shuffle, it transposes data at the bit level, which pairs well with fast duplicate-string-elimination codecs like LZF/LZ4 rather than entropy coders like GZIP.

## Related tools

- **fits2hdf** (CLI utility used with the '-c' compression option to write arrays through HDF5 filters (LZF_SS, bitshuffle, GZIP) and to convert to/from FITS) — https://github.com/telegraphic/fits2hdf
- **bitshuffle** (HDF5 filter (number 32008) that bit-transposes typed binary data before LZF/LZ4 compression; the fastest, highest-ratio codec in the benchmark) — https://github.com/kiyo-masui/bitshuffle
- **CFITSIO** (Library underlying FPACK, used to apply FITS Rice compression for the FITS baseline)
- **hickle** (HDF5-based dump utility that exposes h5py compression keywords (lzf, gzip, shuffle, chunks) usable for compressed storage comparisons) — https://github.com/telegraphic/hickle
- **Python** (Driver language for generating arrays, timing compression, computing ratios, and writing the CSV table)

## Examples

```
fits2hdf input_dir output_dir -c bitshuffle; python -c "import time,os; t=time.time(); os.system('fpack -r in.fits'); print(os.path.getsize('in.fits.fz'), time.time()-t)"
```

## Evaluation signals

- Each (algorithm, N, image-size) cell reproduces the reported Table 1 value within a small numerical tolerance, e.g. bitshuffle N=7, 2048x2048 gives ratio ~3.84 and ~0.04 s
- Compression ratio is consistently computed as uncompressed size / compressed size across all cells
- Compression ratios never exceed the theoretical ceiling implied by the bit range N (e.g. ~3.8x for 7-bit integers in 32-bit containers)
- Decompressed arrays are bit-identical to the originals (lossless round-trip via hdf2fits/fits2hdf)
- bitshuffle shows both higher ratio and lower time than FITS Rice and GZIP across the grid, matching the paper's conclusion

## Limitations

- Results are specific to uniformly distributed random integers; bitshuffle gains depend on inter-element correlation and at least 24-element span correlations
- The paper did not compare lossy compression algorithms, so conclusions do not extend to lossy codecs
- Read/write throughput is dominated by disk I/O; bitshuffle+LZ4 time 'is insignificant compared to the time required to read or write the compressed data to disk'
- Absolute timings are machine-dependent (bitshuffle throughput depends on SSE/AVX support, roughly 1-2 bytes per clock cycle on modern x86)
- HDF5 offers no equivalent to FITS subtractive dithering, which can affect FITS-side compression of certain data

## Evidence

- [results] bitshuffle achieved a compression ratio of 3.84 in 0.04 s: [evidence span withheld — anchor status: not_found]
- [readme] bitshuffle filter number and algorithm: [evidence span withheld — anchor status: external_doc]
- [readme] bitshuffle pairs with LZF/LZ4 not GZIP: [evidence span withheld — anchor status: external_doc]
- [discussion] no lossy comparison performed: [evidence span withheld — anchor status: not_found]
