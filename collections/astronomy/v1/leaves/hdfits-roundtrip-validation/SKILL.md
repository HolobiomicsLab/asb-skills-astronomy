---
name: hdfits-roundtrip-validation
description: Use when converting FITS or MeasurementSet data into HDF5/HDFITS (or
  back) and you must verify the converted file is scientifically identical — e.g.
  before adopting HDF5 storage or benchmarking compression filters against the FITS
  baseline.
when_to_use_negative:
- The data are already stored as HDFITS and no FITS backwards-compatibility check
  is needed.
- You intend to apply genuinely lossy compression and expect bit-identical round-trip
  output — the paper's round-trip equivalence applies to the lossless HDF5 filter
  pipeline.
- Your goal is benchmarking HDF5 lossless filters against FPACK scaled-Rice on floating-point
  data only; that comparison requires a separate dithering-quantization filter, which
  the paper notes HDF5 'currently has no equivalent to'.
edam_operation: http://edamontology.org/operation_0335
tools:
- name: fits2hdf
  role: 'Core converter: read_fits/export_hdf produce HDFITS files, hdf2fits/read_hdf/export_fits
    exercise the reverse path for round-trip validation'
  repo: https://github.com/telegraphic/fits2hdf
- name: h5py
  role: Pythonic HDF5 interface used to write datasets with the compression filter
    pipeline (lzf/gzip/bitshuffle, shuffle, fletcher32) and to compare reloaded arrays
- name: bitshuffle
  role: HDF5 dynamic filter (number 32008) applied during export to test bit-level
    reordering + LZ4/ZSTD compression within the round trip
  repo: https://github.com/kiyo-masui/bitshuffle
- name: CFITSIO
  role: De facto standard library underlying FITS reading/writing for the FITS side
    of the conversion
- name: hickle
  role: Demonstrates the HDF5 filter keyword interface (compression, chunks, shuffle,
    fletcher32) reusable when tuning the export pipeline
  repo: https://github.com/telegraphic/hickle
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ascom.2015.05.001
    title: 'HDFITS: Porting the FITS data model to HDF5'
  source_content_hash: sha256:c296d3b2bf420a39f69b857c5044c11c8185e88025856f910e5d8278adf5ea99
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/telegraphic/fits2hdf
  tools:
  - CFITSIO
  - Python
  - bitshuffle
  - fits2hdf
  - h5py
  - hickle
  edam_operation: http://edamontology.org/operation_0335
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# hdfits-roundtrip-validation

## Summary

Validate that astronomical data survives a FITS-to-HDFITS (HDF5) round trip by converting with fits2hdf's read_fits/export_hdf and read_hdf/export_fits paths and checking that datasets, headers, and (under lossless compression filters such as lzf, gzip, or bitshuffle) pixel values are reproduced exactly.

## When to use

Use when converting FITS or MeasurementSet data into HDF5/HDFITS (or back) and you must verify the converted file is scientifically identical — e.g. before adopting HDF5 storage or benchmarking compression filters against the FITS baseline.

## When NOT to use

- The data are already stored as HDFITS and no FITS backwards-compatibility check is needed.
- You intend to apply genuinely lossy compression and expect bit-identical round-trip output — the paper's round-trip equivalence applies to the lossless HDF5 filter pipeline.
- Your goal is benchmarking HDF5 lossless filters against FPACK scaled-Rice on floating-point data only; that comparison requires a separate dithering-quantization filter, which the paper notes HDF5 'currently has no equivalent to'.

## Inputs

- FITS file (image or binary table) readable with astropy/CFITSIO
- MeasurementSet (MS) convertible via fits2hdf
- In-memory IDI interchange object produced by fits2hdf.io.fitsio.read_fits
- HDF5/HDFITS file with chosen filter pipeline (none/lzf/gzip/bitshuffle, shuffle, fletcher32)

## Outputs

- HDF5 file in HDFITS format (export_hdf output)
- Re-converted FITS file (hdf2fits/export_fits output)
- Round-trip validation result (array equality or per-pixel quantization error metric)
- Compression-ratio metrics (uncompressed bytes / compressed bytes) per filter

## How to apply

Map the source FITS/MS/HDF data into fits2hdf's in-memory interchange format (IDI), then export to HDF5 with fits2hdf.io.hdfio.export_hdf; convert back with hdf2fits or read_hdf/export_fits to exercise the reverse path. Apply the compression options under test when exporting (None, lzf, gzip, bitshuffle — bitshuffle is HDF5 filter 32008, optionally with LZ4/ZSTD post-compression), plus -S shuffle and -C fletcher32 checksums where relevant. Because HDF5 compression filters in this pipeline are lossless, a correct round trip must reproduce the original array bit-for-bit, whereas lossy options (e.g. scaleoffset/dithering quantization) require an explicit error budget instead. Also verify header fidelity: fits2ndf-style conversion stores FITS header cards as-is (as strings), and Python consumers can transparently swap 'from astropy.io import fits' for 'from fits2hdf import pyhdfits as fits' to confirm read equivalence. Record per-method compressed file sizes to compute compression ratios (uncompressed bytes / compressed bytes) as part of the validation report.

## Related tools

- **fits2hdf** (Core converter: read_fits/export_hdf produce HDFITS files, hdf2fits/read_hdf/export_fits exercise the reverse path for round-trip validation) — https://github.com/telegraphic/fits2hdf
- **h5py** (Pythonic HDF5 interface used to write datasets with the compression filter pipeline (lzf/gzip/bitshuffle, shuffle, fletcher32) and to compare reloaded arrays)
- **bitshuffle** (HDF5 dynamic filter (number 32008) applied during export to test bit-level reordering + LZ4/ZSTD compression within the round trip) — https://github.com/kiyo-masui/bitshuffle
- **CFITSIO** (De facto standard library underlying FITS reading/writing for the FITS side of the conversion)
- **hickle** (Demonstrates the HDF5 filter keyword interface (compression, chunks, shuffle, fletcher32) reusable when tuning the export pipeline) — https://github.com/telegraphic/hickle

## Examples

```
from fits2hdf.io.fitsio import read_fits
from fits2hdf.io.hdfio import export_hdf
idi = read_fits('input.fits')
export_hdf(idi, 'output.h5', compression='lzf')
```

## Evaluation signals

- Re-reading the HDFITS file and the re-converted FITS file yields arrays identical to the original (np.all on loaded vs. original pixels) for lossless filters (None/lzf/gzip/bitshuffle).
- Compressed HDF5 file size differs from uncompressed as expected, giving a compression ratio = uncompressed bytes / compressed bytes consistent across repeated runs.
- FITS header cards are preserved as-is (strings) after the round trip, matching the source file's headers.
- Downstream Python code runs unchanged after swapping 'from astropy.io import fits' to 'from fits2hdf import pyhdfits as fits', confirming transparent in-memory FITS equivalence.
- File-size sanity check via os.path.getsize shows the HDFITS output is non-trivially smaller than the input FITS file when a compression filter is enabled.

## Limitations

- The paper compared only lossless compression; 'we have not compared lossy compression algorithms', so round-trip bit-equality claims do not extend to lossy/quantizing filters.
- 'there is currently no equivalent to the subtractive dithering' in HDF5, so FPACK-style lossy float comparison needs a custom registered filter.
- Extending the HDFITS data model requires community input and has not yet occurred; unusual FITS constructs may not survive conversion.
- HDFITS lacks support for columns with masked values and a specification for uncertainties.
- HDF5 is not well supported by data reduction software packages and image viewers, limiting where validated HDFITS files can be consumed.

## Evidence

- [discussion] lossless scope of compression comparison: [evidence span withheld — anchor status: not_found]
