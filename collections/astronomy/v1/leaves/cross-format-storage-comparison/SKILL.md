---
name: cross-format-storage-comparison
description: Use when comparing storage codecs (e.g. FITS Rice vs HDF5 LZF_SS vs bitshuffle)
  on large binary-table or image data files, and you need per-codec compression ratio,
  compression time, and read time with lossless round-trip verification.
when_to_use_negative:
- Evaluating lossy compression algorithms — the underlying study explicitly did not
  compare lossy compression methods, so this skill only supports lossless codecs
- Input files are small or dominated by headers/metadata rather than large typed binary
  arrays, where compression ratios and codec throughput differences are not meaningful
- Data contain correlations spanning fewer than 24 elements — bitshuffle relies on
  neighbouring-element correlation, so ratios will be poor
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: fits2hdf
  role: Converts FITS binary tables to HDF5 (HDFITS format) with '-c lzf' (LZF_SS)
    or '-c bitshuffle' codec options to produce the comparison candidates; also hdf2fits
    for round-trip validation
  repo: https://github.com/telegraphic/fits2hdf
- name: FPACK
  role: Produces the FITS Rice compression baseline via the -table option on binary
    tables
- name: bitshuffle
  role: HDF5 dynamically loaded filter (ID 32008) that bit-transposes typed data in
    ~8 kB blocks before a follow-on codec (LZF/LZ4/ZSTD); required for the '-c bitshuffle'
    fits2hdf run
  repo: https://github.com/kiyo-masui/bitshuffle
- name: h5py
  role: Pythonic interface used by fits2hdf to write chunked, filtered HDF5 datasets
  repo: http://www.h5py.org/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.05.001
    title: 'HDFITS: Porting the FITS data model to HDF5'
  source_content_hash: sha256:8d213202ac2f08a700e66006881895880b0b6e0bfc115076a164a4130a60c6ab
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cross-format-storage-comparison

## Summary

Benchmarking compression ratio and wall-clock time of FITS Rice (FPACK -table) against HDF5 LZF_SS and bitshuffle codecs on large astronomical binary-table files, using fits2hdf for FITS-to-HDF5 conversion and lossless round-trip validation.

## When to use

Use when comparing storage codecs (e.g. FITS Rice vs HDF5 LZF_SS vs bitshuffle) on large binary-table or image data files, and you need per-codec compression ratio, compression time, and read time with lossless round-trip verification.

## When NOT to use

- Evaluating lossy compression algorithms — the underlying study explicitly did not compare lossy compression methods, so this skill only supports lossless codecs
- Input files are small or dominated by headers/metadata rather than large typed binary arrays, where compression ratios and codec throughput differences are not meaningful
- Data contain correlations spanning fewer than 24 elements — bitshuffle relies on neighbouring-element correlation, so ratios will be poor

## Inputs

- Large (>500 MB) binary-table FITS files (e.g. SDSS DR-12 photoMatch Plate-dr12.fits, seguets Obj Set All Dup-0338-3138-0101.fits, sspp Out-dr12.fits)
- FPACK tool with -table option
- fits2hdf utility with -c lzf and -c bitshuffle options and bitshuffle HDF5 filter installed

## Outputs

- compression_benchmark.csv tabulating compression ratio, compression time, and read time for each file-codec combination (3 files x 3 codecs = 9 rows)
- Per-codec compression ratios comparable to reported baselines (2.22 Rice, 2.56 LZF_SS, 1.51 bitshuffle on photoMatch Plate-dr12.fits)
- Lossless round-trip verification for each file-codec combination

## How to apply

Acquire the large (>500 MB) binary-table FITS inputs — in the reference study, three SDSS DR-12 tables: photoMatch Plate-dr12.fits, seguets Obj Set All Dup-0338-3138-0101.fits, and sspp Out-dr12.fits. For each file, run FPACK with the -table option to obtain the FITS Rice baseline ratio and wall-clock compression time; then convert with fits2hdf using '-c lzf' for the HDF5 LZF_SS codec and '-c bitshuffle' (bitshuffle HDF5 filter, filter ID 32008, installed as a dynamically loaded filter) for the bitshuffle codec. Compute compression ratio as uncompressed input size divided by compressed output size, and record compression and read times for all nine file-codec combinations. The rationale for bitshuffle is that it rearranges typed binary data at the bit level (analogous to HDF5's shuffle filter but bit-wise, in ~8 kB blocks) so simple duplicate-string-elimination codecs like LZF, LZ4 and ZSTD outperform entropy coders like GZIP; LZF_SS generally gives higher ratios while bitshuffle is fastest. Validate by checking measured ratios against reported values (e.g. 2.22 for FPACK -table Rice, 2.56 for LZF_SS, 1.51 for bitshuffle on photoMatch Plate-dr12.fits) within reasonable tolerance and confirming every file-codec combination round-trips losslessly.

## Related tools

- **fits2hdf** (Converts FITS binary tables to HDF5 (HDFITS format) with '-c lzf' (LZF_SS) or '-c bitshuffle' codec options to produce the comparison candidates; also hdf2fits for round-trip validation) — https://github.com/telegraphic/fits2hdf
- **FPACK** (Produces the FITS Rice compression baseline via the -table option on binary tables)
- **bitshuffle** (HDF5 dynamically loaded filter (ID 32008) that bit-transposes typed data in ~8 kB blocks before a follow-on codec (LZF/LZ4/ZSTD); required for the '-c bitshuffle' fits2hdf run) — https://github.com/kiyo-masui/bitshuffle
- **h5py** (Pythonic interface used by fits2hdf to write chunked, filtered HDF5 datasets) — http://www.h5py.org/

## Examples

```
fpack -table photoMatchPlate-dr12.fits; fits2hdf . out -c lzf; fits2hdf . out -c bitshuffle
```

## Evaluation signals

- Measured compression ratio on photoMatch Plate-dr12.fits reproduces reported values within reasonable tolerance: 2.22 for FPACK -table Rice, 2.56 for LZF_SS, and 1.51 for bitshuffle
- Every file-codec combination round-trips losslessly (data recovered from compressed output are bit-identical to the original FITS table contents)
- compression_benchmark.csv contains all nine file-codec combinations with ratio, compression time, and read time populated
- Codec ordering is consistent with the reference finding: LZF_SS compresses faster than FITS Rice with a higher ratio, bitshuffle is fastest but with only a modest ratio
- Compressed output sizes are smaller than the uncompressed input sizes for all codecs (ratio > 1.0)

## Limitations

- No comparison of lossy compression algorithms was performed; the benchmark covers only lossless codecs
- HDF5 has no equivalent to FITS subtractive dithering, which can affect compression comparability for some data
- Bitshuffle requires correlations spanning at least 24 dataset elements to improve compression; it does not compress by itself and needs a paired codec (LZF, LZ4 or ZSTD)
- bitshuffle filter must be installed as a dynamically loaded HDF5 filter (ID 32008) or '-c bitshuffle' conversions will fail

## Evidence

- [results] On the three SDSS DR-12 binary tables, LZF_SS compressed faster than FITS Rice with a higher compression ratio, while bitshuffle was fastest but achieved only a modest compression ratio.: [evidence span withheld — anchor status: not_found]
- [supplementary] FPACK -table Rice baseline and fits2hdf codec options: [evidence span withheld — anchor status: not_found]
- [supplementary] Reported benchmark ratios used as validation targets: [evidence span withheld — anchor status: not_found]
- [readme] Bitshuffle algorithm description: [evidence span withheld — anchor status: external_doc]
- [discussion] No lossy comparison performed: [evidence span withheld — anchor status: not_found]
