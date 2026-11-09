---
name: fpack-rice-table-compression
description: Use when comparing lossless compression ratios and I/O throughput of
  FITS Rice (FPACK -table) versus HDF5 (LZF_SS, bitshuffle) codecs on large astronomical
  binary-table datasets, or when deciding whether to migrate FITS tables to HDFITS.
when_to_use_negative:
- The goal involves lossy compression — the article explicitly notes 'we have not
  compared lossy compression algorithms', so this skill only supports lossless codecs.
- The input is not a binary-table FITS file (e.g. image FITS or non-astronomical data),
  since the FPACK -table Rice benchmark and the reported ratios (2.22/2.56/1.51) apply
  to binary tables.
- Exact reproduction of the paper's absolute timing numbers is required on modern
  hardware, since wall-clock times are machine- and I/O-dependent; only ratios should
  be expected to reproduce within tolerance.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3366
tools:
- name: FPACK/CFITSIO
  role: Produces the FITS Rice compression baseline via the -table option on each
    binary-table FITS file
- name: fits2hdf
  role: Converts the FITS tables to HDFITS HDF5 files with -c lzf (LZF_SS) and -c
    bitshuffle codecs for the HDF5 side of the benchmark; hdf2fits converts back for
    lossless round-trip validation
  repo: https://github.com/telegraphic/fits2hdf
- name: bitshuffle
  role: HDF5 dynamically loaded filter (filter number 32008) that bit-transposes data
    before compression; installed to enable fits2hdf -c bitshuffle
  repo: https://github.com/kiyo-masui/bitshuffle
- name: h5py
  role: Pythonic interface to the HDF5 datasets written by fits2hdf, used for verification
    and size/time measurement
- name: Python
  role: Drives the benchmark harness, computes compression ratios (uncompressed size
    / compressed size), and writes compression_benchmark.csv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.05.001
    title: 'HDFITS: Porting the FITS data model to HDF5'
  source_content_hash: sha256:18605b541fc695582c032125748ce8e1d01f3025c0ad9e62bdd408eb246b44ae
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/telegraphic/fits2hdf
  tools:
  - FPACK
  - FPACK/CFITSIO
  - Python
  - bitshuffle
  - fits2hdf
  - h5py
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# fpack-rice-table-compression

## Summary

Benchmark FITS Rice table compression (CFITSIO FPACK with the -table option) against HDF5-based codecs (fits2hdf LZF_SS and bitshuffle) on large binary-table FITS files, measuring compression ratio and wall-clock compression/read times. Grounded in the HDFITS (Price et al. 2015) Table 2 comparison on SDSS DR-12 binary tables.

## When to use

Use when comparing lossless compression ratios and I/O throughput of FITS Rice (FPACK -table) versus HDF5 (LZF_SS, bitshuffle) codecs on large astronomical binary-table datasets, or when deciding whether to migrate FITS tables to HDFITS.

## When NOT to use

- The goal involves lossy compression — the article explicitly notes 'we have not compared lossy compression algorithms', so this skill only supports lossless codecs.
- The input is not a binary-table FITS file (e.g. image FITS or non-astronomical data), since the FPACK -table Rice benchmark and the reported ratios (2.22/2.56/1.51) apply to binary tables.
- Exact reproduction of the paper's absolute timing numbers is required on modern hardware, since wall-clock times are machine- and I/O-dependent; only ratios should be expected to reproduce within tolerance.

## Inputs

- Large (>500 MB) binary-table FITS files from SDSS DR-12 (photoMatch Plate-dr12.fits, seguets Obj Set All Dup-0338-3138-0101.fits, sspp Out-dr12.fits)
- FPACK/CFITSIO command-line tools
- fits2hdf with LZF_SS and bitshuffle HDF5 filter support

## Outputs

- compression_benchmark.csv tabulating compression ratio, compression time, and read time for all three files across FPACK Rice, LZF_SS, and bitshuffle
- Compressed FITS (.fz) and HDF5 (HDFITS) output files for each file-codec combination
- Lossless round-trip verification results (hdf2fits output identical to source FITS)

## How to apply

Download the large (>500 MB) binary-table FITS files (e.g. photoMatch Plate-dr12.fits, seguets Obj Set All Dup-0338-3138-0101.fits, sspp Out-dr12.fits) from the SDSS Science Archive. Run FPACK with the -table option on each file to obtain the FITS Rice compression ratio and wall-clock compression time; this uses CFITSIO's Rice codec, the de facto standard for FITS table compression. Then convert each file with fits2hdf using -c lzf (LZF_SS) and -c bitshuffle (with the bitshuffle HDF5 filter, filter number 32008, installed) to obtain HDF5-side ratios and times. For each of the nine file-codec combinations compute compression ratio as uncompressed size divided by compressed output size, and record compression and read times. Validate by checking measured ratios against the reported values (2.22 for FPACK -table Rice, 2.56 for LZF_SS, 1.51 for bitshuffle on photoMatch Plate-dr12.fits) within tolerance and by round-tripping every combination losslessly (fits2hdf then hdf2fits).

## Related tools

- **FPACK/CFITSIO** (Produces the FITS Rice compression baseline via the -table option on each binary-table FITS file)
- **fits2hdf** (Converts the FITS tables to HDFITS HDF5 files with -c lzf (LZF_SS) and -c bitshuffle codecs for the HDF5 side of the benchmark; hdf2fits converts back for lossless round-trip validation) — https://github.com/telegraphic/fits2hdf
- **bitshuffle** (HDF5 dynamically loaded filter (filter number 32008) that bit-transposes data before compression; installed to enable fits2hdf -c bitshuffle) — https://github.com/kiyo-masui/bitshuffle
- **h5py** (Pythonic interface to the HDF5 datasets written by fits2hdf, used for verification and size/time measurement)
- **Python** (Drives the benchmark harness, computes compression ratios (uncompressed size / compressed size), and writes compression_benchmark.csv)

## Examples

```
fpack -table photoMatch_Plate-dr12.fits; fits2hdf -c lzf photoMatch_Plate-dr12.fits plate_lzf.h5; fits2hdf -c bitshuffle photoMatch_Plate-dr12.fits plate_bshuf.h5; hdf2fits plate_bshuf.h5 plate_rt.fits
```

## Evaluation signals

- Measured compression ratio for FPACK -table Rice on photoMatch Plate-dr12.fits is ~2.22 (within a reasonable tolerance)
- Measured compression ratio for LZF_SS on photoMatch Plate-dr12.fits is ~2.56, and for bitshuffle ~1.51
- Every file-codec combination (3 files x 3 codecs = 9 combinations) round-trips losslessly (hdf2fits output matches the original FITS data)
- Ranking reproduced: LZF_SS compresses faster than FITS Rice with a higher ratio; bitshuffle is fastest but achieves only a modest compression ratio
- compression_benchmark.csv contains ratio, compression time, and read time for all nine file-codec combinations with no missing entries

## Limitations

- No comparison of lossy compression algorithms was performed — the skill covers only lossless codecs (Rice, LZF_SS, bitshuffle).
- HDF5 currently has no equivalent to the FITS subtractive dithering technique, which can affect achievable compression on some data.
- Full characterization of the effect of data access pattern upon read performance is beyond the scope of the paper, so read-time results depend on access patterns used in the benchmark.
- Bitshuffle relies on neighbouring elements being highly correlated; correlations spanning fewer than 24 elements cannot be exploited, and its compression ratio is modest compared to LZF_SS on these tables.

## Evidence

- [results] Rice baseline ratio on SDSS binary tables: [evidence span withheld — anchor status: not_found]
- [discussion] Lossy compression out of scope: [evidence span withheld — anchor status: not_found]
