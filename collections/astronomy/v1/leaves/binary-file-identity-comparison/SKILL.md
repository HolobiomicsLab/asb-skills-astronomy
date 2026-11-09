---
name: binary-file-identity-comparison
description: Use when a file has been converted to another serialization format and
  back (e.g. FITS through HDFITS), and you must prove whether data values survived
  exactly and whether the output is byte-identical.
when_to_use_negative:
- Input has not undergone a lossy-risk conversion — a plain byte checksum (cmp/md5)
  of identical-format copies is sufficient
- You only care about scientific data values and not header provenance, comments,
  or byte-level archival fidelity
- The HDFITS file was written with lossy compression options, where data-value equality
  itself cannot be expected
edam_operation: http://edamontology.org/operation_2409
tools:
- name: fits2hdf
  role: Converts the original FITS file to the HDFITS (HDF5) format that starts the
    round trip
  repo: https://github.com/telegraphic/fits2hdf
- name: hdf2fits
  role: Converts the HDFITS file back to FITS, emitting the round-tripped file to
    be compared against the original
  repo: https://github.com/telegraphic/fits2hdf
- name: Python
  role: Loads both FITS files with astropy.io.fits (or fits2hdf.pyhdfits) and compares
    data-unit arrays element-wise with numpy
- name: HDF5
  role: Serialization format of the intermediate HDFITS file used in the round trip
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2015.05.001
    title: 'HDFITS: Porting the FITS data model to HDF5'
  source_content_hash: sha256:a91a43fbd9c767138ca68bdac1c8a243f642de24855614b3ec238073d72ce1bb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/telegraphic/fits2hdf
  tools:
  - HDF5
  - Python
  - fits2hdf
  - hdf2fits
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# binary-file-identity-comparison

## Summary

Verifies whether a format round-trip (here FITS → HDFITS → FITS via fits2hdf/hdf2fits) is lossless: element-wise comparison of HDU data arrays with numpy, plus a header-card diff to document byte-level re-serialization differences.

## When to use

Use when a file has been converted to another serialization format and back (e.g. FITS through HDFITS), and you must prove whether data values survived exactly and whether the output is byte-identical.

## When NOT to use

- Input has not undergone a lossy-risk conversion — a plain byte checksum (cmp/md5) of identical-format copies is sufficient
- You only care about scientific data values and not header provenance, comments, or byte-level archival fidelity
- The HDFITS file was written with lossy compression options, where data-value equality itself cannot be expected

## Inputs

- Original FITS file (e.g. an SDSS FITS file)
- HDFITS/HDF5 file produced by fits2hdf
- Round-tripped FITS file produced by hdf2fits

## Outputs

- roundtrip_fits.fits (round-tripped FITS file)
- roundtrip_comparison_report.txt documenting data equality and header diffs
- Element-wise numpy comparison result (data equal / not equal) and header-card diff listing formatting-only changes

## How to apply

Run the reverse-conversion tool (hdf2fits input_dir output_dir) with the same options used by fits2hdf to produce a round-tripped file such as roundtrip_fits.fits. Load both the original FITS file and the round-tripped file with astropy.io.fits (or fits2hdf.pyhdfits) and compare every data-unit array element-wise with numpy (e.g. np.array_equal per HDU) to confirm data preservation. Then diff the header cards between the two files to characterize byte-level differences such as added HISTORY cards, stripped mandatory-card comments, and normalized unit strings. Write the findings (data equal, headers not byte-identical) to a comparison report; this matches the paper's caution that fits2hdf round-tripping is not byte-lossless and should not be used for archiving without careful input/output comparison.

## Related tools

- **fits2hdf** (Converts the original FITS file to the HDFITS (HDF5) format that starts the round trip) — https://github.com/telegraphic/fits2hdf
- **hdf2fits** (Converts the HDFITS file back to FITS, emitting the round-tripped file to be compared against the original) — https://github.com/telegraphic/fits2hdf
- **Python** (Loads both FITS files with astropy.io.fits (or fits2hdf.pyhdfits) and compares data-unit arrays element-wise with numpy)
- **HDF5** (Serialization format of the intermediate HDFITS file used in the round trip)

## Examples

```
hdf2fits input_dir output_dir
```

## Evaluation signals

- Every data-unit value in the round-tripped FITS file equals the corresponding value in the original (numpy element-wise equality for each HDU)
- Header diff shows formatting-only changes (added HISTORY cards, stripped mandatory-card comments, normalized unit strings) consistent with hdf2fits re-serialization rather than data loss
- The header-card diff documents concrete byte-level differences rather than asserting identity
- The comparison report explicitly states both the data-equality result and the non-byte-identity of headers
- Files are NOT byte-identical — treating byte equality as the acceptance criterion would incorrectly fail a data-preserving round trip

## Limitations

- fits2hdf round-tripping is not byte-lossless, so the tool cautions it should not be used for archiving without careful comparison of input and output data
- Byte-level differences (HISTORY cards, stripped mandatory-card comments, normalized unit strings) persist even when all HDU data values are unchanged
- HDFITS lacks support for columns of data with masked values, which could complicate comparisons for such data

## Evidence

- [methods] hdf2fits converts HDFITS back to FITS with similar options: "To convert back into FITS, run ``hdf2fits``, which uses similar options::"
- [methods] fits2hdf CLI usage pattern: "fits2hdf input_dir output_dir <options>"
- [methods] fits2hdf maps data through an in-memory interchange format before writing HDF5: "The ``fits2hdf`` utility works by first mapping data from FITS/MS/HDF into an in-memory interchange format (IDI)."
- [readme] fits2hdf purpose as stated in the README: "fits2hdf is a conversion to utility to port FITS files to Hierarchical Data Format (HDF5) files in the HDFITS format."
- [results] Round-trip caveat supporting the comparison requirement: [evidence span withheld — anchor status: not_found]
