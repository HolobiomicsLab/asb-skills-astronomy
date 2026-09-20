---
name: compression-ratio-timing-analysis
description: Use when you need to decide which lossless codec (Rice, LZF, bitshuffle)
  to store large binary-table or array data in, and require reproducible compression-ratio
  and timing numbers per file-codec combination.
when_to_use_negative:
- The data are small (<500 MB) or text/serialised formats where compression timing
  is dominated by I/O noise rather than codec behavior
- Lossy compression is acceptable and should be compared too — the source article
  explicitly notes 'we have not compared lossy compression algorithms'
- The goal is format conversion alone rather than a quantitative codec comparison
  (fits2hdf without timing instrumentation suffices)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_0219
tools:
- name: FPACK
  role: Produces the FITS Rice compression baseline on binary tables using the -table
    option; yields ratio and wall-clock time
- name: fits2hdf
  role: Converts each FITS file to HDFITS with -c lzf (LZF_SS) or -c bitshuffle to
    produce the HDF5 codec arms of the benchmark
  repo: https://github.com/telegraphic/fits2hdf
- name: bitshuffle
  role: HDF5 filter (number 32008) that bit-transposes typed binary data before LZF/LZ4
    compression, used for the bitshuffle arm
  repo: https://github.com/kiyo-masui/bitshuffle
- name: h5py
  role: Pythonic interface used by fits2hdf to write/read the HDF5 datasets during
    conversion and timing runs
- name: Python
  role: Scripting environment for computing ratios (uncompressed/compressed sizes),
    timing, and writing the CSV table
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.05.001
    title: 'HDFITS: Porting the FITS data model to HDF5'
  source_content_hash: sha256:e4d0ace513e31900c1dff57b47e8aeeae26c3a5801f91821c4653a14a7091f3e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/telegraphic/fits2hdf
  tools:
  - FPACK
  - Python
  - bitshuffle
  - fits2hdf
  - h5py
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# compression-ratio-timing-analysis

## Summary

Benchmarks lossless compression codecs (FITS Rice via FPACK -table, HDF5 LZF_SS, HDF5 bitshuffle) on large binary-table files by measuring compression ratio and wall-clock compression/read times, as done in the HDFITS paper on SDSS DR-12 FITS tables.

## When to use

Use when you need to decide which lossless codec (Rice, LZF, bitshuffle) to store large binary-table or array data in, and require reproducible compression-ratio and timing numbers per file-codec combination.

## When NOT to use

- The data are small (<500 MB) or text/serialised formats where compression timing is dominated by I/O noise rather than codec behavior
- Lossy compression is acceptable and should be compared too — the source article explicitly notes 'we have not compared lossy compression algorithms'
- The goal is format conversion alone rather than a quantitative codec comparison (fits2hdf without timing instrumentation suffices)

## Inputs

- Large (>500 MB) binary-table FITS files, e.g. SDSS DR-12 photoMatch Plate-dr12.fits
- SDSS DR-12 seguets Obj Set All Dup-0338-3138-0101.fits
- SDSS DR-12 sspp Out-dr12.fits
- Installed FPACK (CFITSIO-based) with -table support
- fits2hdf with h5py, HDF5, and the bitshuffle filter plugin

## Outputs

- Compression ratio (uncompressed size / compressed output size) per file-codec combination
- Wall-clock compression time and read time per file-codec combination
- Tabulated benchmark results, e.g. compression_benchmark.csv

## How to apply

Acquire the large (>500 MB) input files (in the reference workflow, three SDSS DR-12 binary-table FITS files: photoMatch Plate-dr12.fits, seguets Obj Set All Dup-0338-3138-0101.fits, sspp Out-dr12.fits). For the FITS baseline run FPACK with the -table option to obtain the Rice compression ratio and wall-clock time; for the HDF5 candidates run fits2hdf with -c lzf (LZF_SS) and with -c bitshuffle (bitshuffle filter installed, HDF5 filter 32008). Compute compression ratio as uncompressed file size divided by compressed output size, and record compression and read times for all file-codec combinations. Tabulate results (e.g. compression_benchmark.csv) and validate round-trip losslessness via hdf2fits/fits2hdf conversion, checking ratios against expected values (2.22 Rice, 2.56 LZF_SS, 1.51 bitshuffle on photoMatch Plate-dr12.fits) within reasonable tolerance.

## Related tools

- **FPACK** (Produces the FITS Rice compression baseline on binary tables using the -table option; yields ratio and wall-clock time)
- **fits2hdf** (Converts each FITS file to HDFITS with -c lzf (LZF_SS) or -c bitshuffle to produce the HDF5 codec arms of the benchmark) — https://github.com/telegraphic/fits2hdf
- **bitshuffle** (HDF5 filter (number 32008) that bit-transposes typed binary data before LZF/LZ4 compression, used for the bitshuffle arm) — https://github.com/kiyo-masui/bitshuffle
- **h5py** (Pythonic interface used by fits2hdf to write/read the HDF5 datasets during conversion and timing runs)
- **Python** (Scripting environment for computing ratios (uncompressed/compressed sizes), timing, and writing the CSV table)

## Examples

```
fpack -table photoMatch Plate-dr12.fits; fits2hdf Plate-dr12.fits Plate-dr12.h5 -c bitshuffle
```

## Evaluation signals

- Measured compression ratios on photoMatch Plate-dr12.fits reproduce the reported 2.22 (FPACK -table Rice), 2.56 (LZF_SS), and 1.51 (bitshuffle) within a reasonable tolerance
- Every file-codec combination round-trips losslessly (fits2hdf then hdf2fits recovers the original FITS data)
- All 9 file-codec combinations (3 files x 3 codecs) appear in the output table with both ratio and timing values
- Compression ratio is consistently computed as uncompressed file size divided by compressed output size across codecs
- Expected qualitative ordering holds: LZF_SS compresses faster than FITS Rice with higher ratio; bitshuffle is fastest but with only modest ratio

## Limitations

- No lossy compression algorithms were compared; results only cover lossless codecs
- Bitshuffle relies on neighbouring dataset elements being highly correlated and exploits correlations spanning at least 24 elements; uncorrelated data compress poorly (bitshuffle reached only 1.51 ratio on photoMatch Plate-dr12.fits)
- HDF5 currently has no equivalent to the FITS subtractive dithering technique, which can limit compression of some FITS table data
- Read-performance conclusions depend on data access patterns; full characterization of access-pattern effects was beyond the paper's scope
- bitshuffle must be installed as an HDF5 dynamically loaded filter (filter 32008) or the -c bitshuffle conversion fails

## Evidence

- [discussion] Lossy compression not compared: [evidence span withheld — anchor status: not_found]
