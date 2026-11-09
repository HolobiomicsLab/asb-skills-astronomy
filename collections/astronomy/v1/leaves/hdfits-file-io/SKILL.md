---
name: hdfits-file-io
description: Use when FITS files (e.g. SDSS images, radio-interferometric data) need
  to be converted to HDF5 for faster reads or compression, or when an HDFITS file
  must be emitted back as FITS for legacy software.
when_to_use_negative:
- Archival storage requiring byte-for-byte reproducibility of the original FITS file
  — fits2hdf round-tripping is not byte-lossless
- Data with masked-value columns that need faithful preservation — HDFITS lacks good
  support for columns with masked values
- Workflows depending on FITS-specific features without HDF5 equivalents, such as
  subtractive dithering
tools:
- name: fits2hdf
  role: Command line tool that converts FITS files to HDF5-based HDFITS format via
    the in-memory IDI interchange format
  repo: https://github.com/telegraphic/fits2hdf
- name: hdf2fits
  role: Companion utility using similar options to convert HDFITS files back into
    FITS
  repo: https://github.com/telegraphic/fits2hdf
- name: Python
  role: Used with astropy.io.fits or fits2hdf.pyhdfits to load both files and compare
    data arrays element-wise with numpy
- name: HDF5
  role: Underlying serialization format (accessed via h5py) that provides chunking,
    compression filters, and faster reads than FITS
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2015.05.001
    title: 'HDFITS: Porting the FITS data model to HDF5'
  source_content_hash: sha256:b3954eeead9509a0e44f35ddc1c7a92fe71bc3d19d0a3a9c0947e8e5050ab197
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/telegraphic/fits2hdf
  tools:
  - HDF5
  - Python
  - fits2hdf
  - hdf2fits
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# hdfits-file-io

## Summary

Round-trip astronomical data between FITS and the HDF5-based HDFITS format using the fits2hdf/hdf2fits utilities, verifying that HDU data values survive the round trip even though files are not byte-identical.

## When to use

Use when FITS files (e.g. SDSS images, radio-interferometric data) need to be converted to HDF5 for faster reads or compression, or when an HDFITS file must be emitted back as FITS for legacy software.

## When NOT to use

- Archival storage requiring byte-for-byte reproducibility of the original FITS file — fits2hdf round-tripping is not byte-lossless
- Data with masked-value columns that need faithful preservation — HDFITS lacks good support for columns with masked values
- Workflows depending on FITS-specific features without HDF5 equivalents, such as subtractive dithering

## Inputs

- FITS files (e.g. SDSS FITS images)
- MeasurementSet (MS) data
- HDFITS/HDF5 files produced by fits2hdf

## Outputs

- HDFITS (.h5/HDF5) files in the HDFITS format
- Round-tripped FITS files (e.g. roundtrip_fits.fits) emitted by hdf2fits
- Comparison report documenting data equality and header diffs (e.g. roundtrip_comparison_report.txt)

## How to apply

Map the source data (FITS/MS/HDF) through fits2hdf's in-memory interchange format (IDI) and write an HDFITS file with the fits2hdf command line tool (fits2hdf input_dir output_dir with compression options such as bitshuffle or gzip). To convert back, run hdf2fits with the same options to produce a round-tripped FITS file. Validate by loading both the original and round-tripped files with astropy.io.fits (or fits2hdf.pyhdfits) and comparing data-unit arrays element-wise with numpy: values should match exactly. Then diff header cards to document re-serialization effects (added HISTORY cards, stripped mandatory-card comments, normalized unit strings) — the paper's own caution is that round-tripping is NOT byte-lossless, so output should be compared carefully before using fits2hdf for archiving.

## Related tools

- **fits2hdf** (Command line tool that converts FITS files to HDF5-based HDFITS format via the in-memory IDI interchange format) — https://github.com/telegraphic/fits2hdf
- **hdf2fits** (Companion utility using similar options to convert HDFITS files back into FITS) — https://github.com/telegraphic/fits2hdf
- **Python** (Used with astropy.io.fits or fits2hdf.pyhdfits to load both files and compare data arrays element-wise with numpy)
- **HDF5** (Underlying serialization format (accessed via h5py) that provides chunking, compression filters, and faster reads than FITS)

## Examples

```
fits2hdf input_dir output_dir && hdf2fits input_dir output_dir
```

## Evaluation signals

- Every data-unit value in the round-tripped FITS file equals the original value when compared element-wise with numpy
- Header diff between original and round-tripped files shows formatting-only changes (added HISTORY cards, stripped mandatory-card comments, normalized unit strings) consistent with re-serialization, not data loss
- The output file is a valid FITS file loadable by astropy.io.fits / CFITSIO-based legacy software
- The HDFITS file is readable in Python by only changing import lines (astropy.io.fits → fits2hdf.pyhdfits)
- Compression and read-speed improvements (paper reports up to 100x faster reads) can be measured against the original FITS baseline

## Limitations

- fits2hdf round-tripping is not byte-lossless; the tool cautions it should not be used for archiving without careful comparison of input and output data
- HDFITS lacks better support for columns of data with masked values and has no specification for addressing uncertainties
- HDF5 has no equivalent to the FITS subtractive dithering technique, and lossy compression algorithms were not compared in the paper
- HDF5 is not well supported by many data reduction software packages and image viewers, requiring conversion back to FITS for legacy tools

## Evidence
