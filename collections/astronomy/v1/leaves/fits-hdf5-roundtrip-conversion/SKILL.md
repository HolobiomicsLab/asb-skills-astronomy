---
name: fits-hdf5-roundtrip-conversion
description: Use when a FITS file must be moved into HDF5 (HDFITS) and back, and you
  need to establish whether the conversion is lossless at the data-unit level versus
  the byte level.
when_to_use_negative:
- Archival storage where byte-identical preservation of the original FITS file is
  required — the tool itself cautions against using fits2hdf for archiving without
  careful input/output comparison
- Data relying on columns of masked values, which HDFITS does not yet support well
- Studies of lossy compression algorithms, which the authors explicitly did not compare
edam_operation: http://edamontology.org/operation_2409
tools:
- name: fits2hdf
  role: Converts FITS files into the HDFITS HDF5 format by mapping data through an
    in-memory interchange format (IDI)
  repo: https://github.com/telegraphic/fits2hdf
- name: hdf2fits
  role: Converts HDFITS back into FITS for the round trip, re-serializing headers
    with formatting-only changes
  repo: https://github.com/telegraphic/fits2hdf
- name: Python
  role: Used with astropy.io.fits and numpy to load both files, compare data-unit
    values element-wise, and diff header cards
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.05.001
    title: 'HDFITS: Porting the FITS data model to HDF5'
  source_content_hash: sha256:85b13bcee35f9cc17f3fc296441a49972e8badb9626b950566a2c1a46440e814
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/telegraphic/fits2hdf
  tools:
  - HDF5
  - Python
  - fits2hdf
  - h5py
  - hdf2fits
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# fits-hdf5-roundtrip-conversion

## Summary

Convert FITS files to the HDFITS HDF5 format with fits2hdf and back with hdf2fits, then verify the round trip by confirming HDU data values are preserved while documenting that headers are not byte-identical.

## When to use

Use when a FITS file must be moved into HDF5 (HDFITS) and back, and you need to establish whether the conversion is lossless at the data-unit level versus the byte level.

## When NOT to use

- Archival storage where byte-identical preservation of the original FITS file is required — the tool itself cautions against using fits2hdf for archiving without careful input/output comparison
- Data relying on columns of masked values, which HDFITS does not yet support well
- Studies of lossy compression algorithms, which the authors explicitly did not compare

## Inputs

- Original FITS file (e.g. an SDSS FITS file) with data units and header cards
- HDFITS/HDF5 file produced by fits2hdf

## Outputs

- Round-tripped FITS file (e.g. roundtrip_fits.fits) emitted by hdf2fits
- Round-trip comparison report (e.g. roundtrip_comparison_report.txt) documenting data equality and header diffs

## How to apply

Map the FITS data through fits2hdf's in-memory interchange format (IDI) with `fits2hdf input_dir output_dir <options>`, then convert back with `hdf2fits input_dir output_dir` using the same options to emit a round-tripped file such as roundtrip_fits.fits. Load both the round-tripped and original files with astropy.io.fits (or fits2hdf.pyhdfits) and compare every data-unit array element-wise with numpy to confirm exact equality. Separately diff the header cards to document formatting-only changes: added HISTORY cards, stripped mandatory-card comments, and normalized unit strings arising from hdf2fits re-serialization. The conversion is validated as successful when all data values match, while the files are expected NOT to be byte-identical, so fits2hdf should not be trusted for archiving without this careful input/output comparison.

## Related tools

- **fits2hdf** (Converts FITS files into the HDFITS HDF5 format by mapping data through an in-memory interchange format (IDI)) — https://github.com/telegraphic/fits2hdf
- **hdf2fits** (Converts HDFITS back into FITS for the round trip, re-serializing headers with formatting-only changes) — https://github.com/telegraphic/fits2hdf
- **Python** (Used with astropy.io.fits and numpy to load both files, compare data-unit values element-wise, and diff header cards)

## Examples

```
fits2hdf input_dir output_dir && hdf2fits output_dir roundtrip_dir
```

## Evaluation signals

- Every data-unit array value in the round-tripped FITS file equals the original value exactly (numpy element-wise comparison)
- Header diff shows formatting-only changes consistent with hdf2fits re-serialization (added HISTORY cards, stripped mandatory-card comments, normalized unit strings) rather than data loss
- Round-tripped file is NOT byte-identical to the original — byte equality would indicate the diff procedure was faulty
- Comparison findings (data equal, headers not byte-identical) are written to the output report file

## Limitations

- fits2hdf round-tripping is not byte-lossless, so it should not be used for archiving without careful comparison of input and output data, even though HDU data values remain unchanged
- HDFITS needs better support for columns of data with masked values
- HDF5 currently has no equivalent to the FITS subtractive dithering technique

## Evidence
