---
name: fits-header-card-diffing
description: Use when a FITS file has been converted to another format (e.g. HDFITS
  via fits2hdf) and back, and you must verify whether HDU data values and header cards
  survived the round trip, or when deciding if a converted file is safe for archiving.
when_to_use_negative:
- When byte-identical archival copies are required with no comparison step — fits2hdf
  round-tripping is explicitly not byte-lossless and the tool cautions against archiving
  without careful comparison
- When the input is not FITS-derived data (e.g. a raw MeasurementSet or native HDF5
  file), since this skill's diff assumes a FITS original and a round-tripped FITS
  output
- When only the HDF5/HDFITS side has changed and no FITS round-trip conversion was
  performed, since there is no second FITS header to diff
tools:
- name: fits2hdf
  role: Converts the original FITS file to HDFITS (mapping FITS/MS/HDF data through
    the IDI interchange format), producing the HDF5 file whose round trip will be
    diffed
  repo: https://github.com/telegraphic/fits2hdf
- name: hdf2fits
  role: Converts the HDFITS file back to FITS (with options similar to fits2hdf),
    emitting the round-tripped FITS file used in the header/data comparison
  repo: https://github.com/telegraphic/fits2hdf
- name: Astropy
  role: astropy.io.fits loads both the original and round-tripped FITS files so HDU
    data arrays and header cards can be compared in Python
- name: Python
  role: Hosts the numpy element-wise data comparison and the header-card diffing logic
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2015.05.001
    title: 'HDFITS: Porting the FITS data model to HDF5'
  source_content_hash: sha256:4c268a19043ce310cb5fcc898ebd41969510f8ca84cd50f21a8ec5456eacd768
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/telegraphic/fits2hdf
  tools:
  - Astropy
  - Python
  - fits2hdf
  - h5py
  - hdf2fits
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# fits-header-card-diffing

## Summary

A verification method for round-trip FITS conversions (e.g. fits2hdf → hdf2fits) that element-wise compares HDU data arrays with numpy and diffs header cards to separate formatting-only re-serialization changes from actual data loss.

## When to use

Use when a FITS file has been converted to another format (e.g. HDFITS via fits2hdf) and back, and you must verify whether HDU data values and header cards survived the round trip, or when deciding if a converted file is safe for archiving.

## When NOT to use

- When byte-identical archival copies are required with no comparison step — fits2hdf round-tripping is explicitly not byte-lossless and the tool cautions against archiving without careful comparison
- When the input is not FITS-derived data (e.g. a raw MeasurementSet or native HDF5 file), since this skill's diff assumes a FITS original and a round-tripped FITS output
- When only the HDF5/HDFITS side has changed and no FITS round-trip conversion was performed, since there is no second FITS header to diff

## Inputs

- Original FITS file (e.g. an SDSS FITS file)
- Round-tripped FITS file (e.g. roundtrip_fits.fits emitted by hdf2fits from an HDFITS .h5 produced by fits2hdf)

## Outputs

- Element-wise numpy comparison result for each HDU data array (equal / not equal)
- Header card diff listing added HISTORY cards, stripped mandatory-card comments, and normalized unit strings
- Text report (e.g. roundtrip_comparison_report.txt) documenting that data values are preserved but files are not byte-identical

## How to apply

Load both the original and round-tripped FITS files with astropy.io.fits (or fits2hdf.pyhdfits), then compare every data-unit array element-wise using numpy (e.g. np.array_equal) to confirm the data values are unchanged. Next, diff the header cards between the two files, documenting the byte-level differences hdf2fits introduces: added HISTORY cards, stripped mandatory-card comments, and normalized unit strings. Classify each difference as formatting-only re-serialization (acceptable) versus data loss (failure). Write the findings — data equal, headers not byte-identical — into a comparison report (e.g. roundtrip_comparison_report.txt). The rationale, per the HDFITS paper, is that fits2hdf round-tripping preserves data values but is not byte-lossless, so the authors caution against using it for archiving without careful input/output comparison.

## Related tools

- **fits2hdf** (Converts the original FITS file to HDFITS (mapping FITS/MS/HDF data through the IDI interchange format), producing the HDF5 file whose round trip will be diffed) — https://github.com/telegraphic/fits2hdf
- **hdf2fits** (Converts the HDFITS file back to FITS (with options similar to fits2hdf), emitting the round-tripped FITS file used in the header/data comparison) — https://github.com/telegraphic/fits2hdf
- **Astropy** (astropy.io.fits loads both the original and round-tripped FITS files so HDU data arrays and header cards can be compared in Python)
- **Python** (Hosts the numpy element-wise data comparison and the header-card diffing logic)

## Examples

```
hdf2fits input_dir output_dir
```

## Evaluation signals

- Every data-unit value in the round-tripped FITS file equals the original value under element-wise numpy comparison (np.array_equal passes for each HDU)
- The header diff shows only formatting-level changes consistent with hdf2fits re-serialization: added HISTORY cards, stripped mandatory-card comments, and normalized unit strings
- No header diff entry indicates changed or missing keyword values that would imply data loss
- Both files open successfully with astropy.io.fits (or fits2hdf.pyhdfits) without warnings about malformed cards
- The written report explicitly states 'data equal, headers not byte-identical', matching the paper's finding that round-tripping preserves values but not bytes

## Limitations

- fits2hdf round-tripping is not byte-lossless, so the tool cautions it should not be used for archiving without careful comparison of input and output data
- Header re-serialization introduces formatting changes (added HISTORY cards, stripped mandatory-card comments, normalized unit strings) that prevent byte-identical output even when data values are preserved
- The comparison detects value preservation, not metadata fidelity — comments and unit-string formatting are expected to change and cannot be treated as errors

## Evidence
