---
name: fits-binary-table-compression-benchmarking
description: Use when large (>500 MB) binary-table FITS files must be compressed or
  converted to HDF5 and you need to compare codec ratios/timings before choosing a
  storage format.
when_to_use_negative:
- Evaluating lossy compression algorithms — the paper explicitly notes 'we have not
  compared lossy compression algorithms'
- Small files or image (non-binary-table) FITS data where chunked-filter overhead
  dominates and the >500 MB SDSS binary-table setup does not apply
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_0219
tools:
- name: fits2hdf
  role: Converts binary-table FITS files to HDFITS HDF5 files with -c lzf or -c bitshuffle
    codecs for benchmarking; hdf2fits verifies lossless round-trip
  repo: https://github.com/telegraphic/fits2hdf
- name: bitshuffle
  role: HDF5 filter (number 32008) that rearranges typed binary data at the bit level
    before compression; installed so fits2hdf -c bitshuffle works
  repo: https://github.com/kiyo-masui/bitshuffle
- name: FPACK
  role: CFITSIO-based tool run with -table to produce the FITS Rice compression baseline
    ratio and timing
- name: h5py
  role: Pythonic HDF5 interface underlying fits2hdf used for programmatic checks of
    compressed datasets
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.05.001
    title: 'HDFITS: Porting the FITS data model to HDF5'
  source_content_hash: sha256:694ec8a97ab05c3d245a32927f00e3c845e0f0a541a347c6d4c1d04eee86e798
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# fits-binary-table-compression-benchmarking

## Summary

Benchmark FITS Rice (FPACK -table) against HDF5 LZF_SS and bitshuffle compression for large binary-table FITS files, measuring compression ratio and compression/read wall-clock time. It grounds format-choice decisions (FITS vs HDFITS) in measured throughput and ratio numbers.

## When to use

Use when large (>500 MB) binary-table FITS files must be compressed or converted to HDF5 and you need to compare codec ratios/timings before choosing a storage format.

## When NOT to use

- Evaluating lossy compression algorithms — the paper explicitly notes 'we have not compared lossy compression algorithms'
- Small files or image (non-binary-table) FITS data where chunked-filter overhead dominates and the >500 MB SDSS binary-table setup does not apply

## Inputs

- Binary-table FITS files >500 MB (e.g. SDSS DR-12 photoMatch Plate-dr12.fits)
- FPACK with -table option
- fits2hdf with -c lzf and -c bitshuffle options (bitshuffle filter installed)

## Outputs

- compression_benchmark.csv with compression ratio and compression/read times for all file-codec combinations
- Compressed outputs: FITS Rice files, HDF5 LZF_SS files, HDF5 bitshuffle HDFITS files

## How to apply

Download three >500 MB SDSS DR-12 binary-table FITS files (e.g. photoMatch Plate-dr12.fits, seguets Obj Set All Dup-0338-3138-0101.fits, sspp Out-dr12.fits). Run FPACK with the -table option on each file to obtain the FITS Rice compression ratio and wall-clock time. Run fits2hdf with '-c lzf' and then with '-c bitshuffle' (bitshuffle HDF5 filter installed) to obtain LZF_SS and bitshuffle ratios and times. Compute the compression ratio as uncompressed file size divided by compressed output size and record compression and read times for all nine file-codec combinations; tabulate results (e.g. compression_benchmark.csv). Validate by checking measured ratios against reported benchmark values (2.22 Rice, 2.56 LZF_SS, 1.51 bitshuffle on photoMatch Plate-dr12.fits) within tolerance and confirming lossless round-trip via hdf2fits.

## Related tools

- **fits2hdf** (Converts binary-table FITS files to HDFITS HDF5 files with -c lzf or -c bitshuffle codecs for benchmarking; hdf2fits verifies lossless round-trip) — https://github.com/telegraphic/fits2hdf
- **bitshuffle** (HDF5 filter (number 32008) that rearranges typed binary data at the bit level before compression; installed so fits2hdf -c bitshuffle works) — https://github.com/kiyo-masui/bitshuffle
- **FPACK** (CFITSIO-based tool run with -table to produce the FITS Rice compression baseline ratio and timing)
- **h5py** (Pythonic HDF5 interface underlying fits2hdf used for programmatic checks of compressed datasets)

## Examples

```
fpack -table photoMatch-Plate-dr12.fits; fits2hdf fits_dir h5_dir -c lzf; fits2hdf fits_dir h5_dir -c bitshuffle
```

## Evaluation signals

- Measured compression ratio on photoMatch Plate-dr12.fits reproduces the reported values within tolerance: 2.22 (FPACK -table Rice), 2.56 (LZF_SS), 1.51 (bitshuffle)
- Every one of the nine file-codec combinations round-trips losslessly (hdf2fits output matches the original FITS data)
- All three SDSS DR-12 input files are >500 MB so ratios are not dominated by fixed header overhead
- Tabulated CSV contains ratio plus compression and read times for all 3 files x 3 codecs
- Relative ordering is qualitatively consistent: LZF_SS compresses faster than Rice with a higher ratio; bitshuffle is fastest but has a modest ratio

## Limitations

- No comparison of lossy compression algorithms was performed
- Full characterization of the effect of data access pattern upon read performance is beyond the scope of the paper
- Bitshuffle achieves only a modest compression ratio on binary tables despite being fastest; it relies on neighbouring elements being highly correlated
- HDF5 is not well supported by data reduction software packages and image viewers, limiting practical adoption of the HDFITS outputs

## Evidence

- [discussion] No lossy compression comparison: [evidence span withheld — anchor status: not_found]
- [readme] Bitshuffle is a bit-level rearrangement filter matched to LZF/LZ4: [evidence span withheld — anchor status: external_doc]
