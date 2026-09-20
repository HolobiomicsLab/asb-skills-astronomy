---
name: hdf5-chunked-lossless-compression
description: Use when large (>500 MB) binary-table FITS files must be compressed losslessly
  and you need to choose between FITS Rice, HDF5 LZF_SS, and HDF5 bitshuffle by measuring
  compression ratio and speed.
when_to_use_negative:
- Data are images or radio interferometric visibilities where bitshuffle is intended
  (good for radio astronomy data) rather than SDSS-style binary tables
- Lossy compression is acceptable — the paper explicitly did not compare lossy compression
  algorithms
- The file must remain directly readable by legacy FITS-only data reduction software
  or image viewers, since HDF5 is not well supported by those packages
edam_operation: http://edamontology.org/operation_0335
tools:
- name: fits2hdf
  role: Converts FITS files into HDFITS-format HDF5 with chosen chunked filter (-c
    lzf for LZF_SS, -c bitshuffle); hdf2fits restores FITS for lossless round-trip
    checks
  repo: https://github.com/telegraphic/fits2hdf
- name: bitshuffle
  role: HDF5 dynamically loaded filter (ID 32008) that bit-transposes typed binary
    data before LZF/LZ4 compression, installed so fits2hdf can use -c bitshuffle
  repo: https://github.com/kiyo-masui/bitshuffle
- name: FPACK
  role: CFITSIO tool run with the -table option to produce the FITS Rice-compressed
    baseline for binary tables
- name: h5py
  role: Pythonic interface used by fits2hdf to write chunked, filter-compressed HDF5
    datasets
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.05.001
    title: 'HDFITS: Porting the FITS data model to HDF5'
  source_content_hash: sha256:a53ea5c013fb837565e2f5c07f1609452680929fca80b3c2f8b738dca88a3c4e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/telegraphic/fits2hdf
  tools:
  - FPACK
  - HDF5
  - Python
  - bitshuffle
  - fits2hdf
  - h5py
  edam_operation: http://edamontology.org/operation_0335
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# hdf5-chunked-lossless-compression

## Summary

Benchmarks lossless compression of large binary-table FITS files by comparing FITS Rice (FPACK -table) against HDF5 chunked filters (LZF_SS and bitshuffle via fits2hdf), quantifying compression ratio and wall-clock time. It exploits the finding that the FITS data model ports straightforwardly to HDF5 with improved compression and faster reads.

## When to use

Use when large (>500 MB) binary-table FITS files must be compressed losslessly and you need to choose between FITS Rice, HDF5 LZF_SS, and HDF5 bitshuffle by measuring compression ratio and speed.

## When NOT to use

- Data are images or radio interferometric visibilities where bitshuffle is intended (good for radio astronomy data) rather than SDSS-style binary tables
- Lossy compression is acceptable — the paper explicitly did not compare lossy compression algorithms
- The file must remain directly readable by legacy FITS-only data reduction software or image viewers, since HDF5 is not well supported by those packages

## Inputs

- Binary-table FITS files (>500 MB), e.g. SDSS DR-12 photoMatch Plate-dr12.fits
- SDSS DR-12 seguets Obj Set All Dup-0338-3138-0101.fits
- SDSS DR-12 sspp Out-dr12.fits

## Outputs

- compression_benchmark.csv tabulating compression ratio and compression/read times for all three files across three codecs
- Rice-compressed FITS files (fpack -table output)
- HDF5/HDFITS files compressed with LZF_SS or bitshuffle filters

## How to apply

Acquire the three SDSS DR-12 binary-table FITS files (photoMatch Plate-dr12.fits, seguets Obj Set All Dup-0338-3138-0101.fits, sspp Out-dr12.fits) from the SDSS Science Archive. Run FPACK with the -table option on each file to get the FITS Rice baseline, then convert each file with fits2hdf using -c lzf (LZF_SS) and -c bitshuffle (with the bitshuffle HDF5 filter, ID 32008, installed), recording compression and read wall-clock times for all nine file-codec combinations. Compute compression ratio as uncompressed file size divided by compressed output size. Validate by checking ratios reproduce the reported benchmark values (2.22 for FPACK -table Rice, 2.56 for LZF_SS, 1.51 for bitshuffle on photoMatch Plate-dr12.fits) within tolerance and that every combination round-trips losslessly back to FITS via hdf2fits.

## Related tools

- **fits2hdf** (Converts FITS files into HDFITS-format HDF5 with chosen chunked filter (-c lzf for LZF_SS, -c bitshuffle); hdf2fits restores FITS for lossless round-trip checks) — https://github.com/telegraphic/fits2hdf
- **bitshuffle** (HDF5 dynamically loaded filter (ID 32008) that bit-transposes typed binary data before LZF/LZ4 compression, installed so fits2hdf can use -c bitshuffle) — https://github.com/kiyo-masui/bitshuffle
- **FPACK** (CFITSIO tool run with the -table option to produce the FITS Rice-compressed baseline for binary tables)
- **h5py** (Pythonic interface used by fits2hdf to write chunked, filter-compressed HDF5 datasets)

## Examples

```
fpack -table photoMatch Plate-dr12.fits && fits2hdf -c bitshuffle input_dir output_dir
```

## Evaluation signals

- Compression ratios reproduce the reported values within reasonable tolerance: 2.22 (FPACK -table Rice), 2.56 (LZF_SS), 1.51 (bitshuffle) on photoMatch Plate-dr12.fits
- Every file-codec combination round-trips losslessly (hdf2fits output matches the original FITS data)
- Wall-clock compression and read times are recorded for all nine file-codec combinations (3 files x 3 codecs)
- HDF5 output uses the expected filter pipeline (bitshuffle filter ID 32008 / LZF) on chunked datasets
- Results are saved as compression_benchmark.csv with ratio and time columns per file and codec

## Limitations

- Bitshuffle achieves only a modest compression ratio (~1.51) on these binary tables despite being fastest; it relies on neighbouring dataset elements being highly correlated
- HDF5/HDFITS is not well supported by data reduction software packages and image viewers, limiting adoption
- Lossy compression algorithms were not compared, and HDF5 currently has no equivalent to the FITS subtractive dithering technique

## Evidence
