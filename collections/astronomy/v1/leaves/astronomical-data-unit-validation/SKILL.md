---
name: astronomical-data-unit-validation
description: Use when a FITS file has been converted to another format (HDF5/HDFITS)
  and converted back, and you must prove whether the round trip is lossless at the
  data-value and header/byte level.
when_to_use_negative:
- When the goal is faster reads or better compression rather than archival byte-fidelity
  verification — fits2hdf's authors caution it is not byte-lossless and should not
  be used for archiving without careful comparison
- When input is not FITS or HDFITS data (e.g. MeasurementSet conversion follows a
  different fits2hdf pathway and the same header-diff validation does not apply)
tools:
- name: fits2hdf
  role: Converts the original FITS file into the HDFITS (HDF5) format as the forward
    leg of the round trip
  repo: https://github.com/telegraphic/fits2hdf
- name: hdf2fits
  role: Converts the HDFITS file back into FITS to produce the round-tripped file
    for comparison
  repo: https://github.com/telegraphic/fits2hdf
- name: Python
  role: Hosts astropy.io.fits / fits2hdf.pyhdfits loading and numpy element-wise data
    comparison plus header diffing
- name: HDF5
  role: Serialization backend of the HDFITS intermediate format used in the round
    trip
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2015.05.001
    title: 'HDFITS: Porting the FITS data model to HDF5'
  source_content_hash: sha256:d30bee5b851090ef8000abb14a97e022f05685e248530f88ba3cd77a20ff5126
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
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# astronomical-data-unit-validation

## Summary

A validation procedure for FITS-derived astronomical data after format conversion (e.g. FITS to HDFITS via fits2hdf and back via hdf2fits): confirm data-unit array values are preserved element-wise while documenting that headers are re-serialized and not byte-identical.

## When to use

Use when a FITS file has been converted to another format (HDF5/HDFITS) and converted back, and you must prove whether the round trip is lossless at the data-value and header/byte level.

## When NOT to use

- When the goal is faster reads or better compression rather than archival byte-fidelity verification — fits2hdf's authors caution it is not byte-lossless and should not be used for archiving without careful comparison
- When input is not FITS or HDFITS data (e.g. MeasurementSet conversion follows a different fits2hdf pathway and the same header-diff validation does not apply)

## Inputs

- Original astronomical FITS file (e.g. an SDSS FITS file)
- HDFITS/HDF5 file produced by fits2hdf
- Round-tripped FITS file produced by hdf2fits

## Outputs

- Element-wise numpy comparison of data-unit arrays (pass/fail per HDU)
- Header card diff documenting byte-level differences (HISTORY cards, stripped comments, unit strings)
- Comparison report (e.g. roundtrip_comparison_report.txt) stating data equal but files not byte-identical

## How to apply

Convert the FITS file to HDFITS with the fits2hdf command line tool (fits2hdf input_dir output_dir <options>), then convert back with hdf2fits, which uses similar options, producing a round-tripped FITS file. Load both the round-tripped file and the original with astropy.io.fits (or fits2hdf.pyhdfits) in Python and compare data-unit array values element-wise with numpy to confirm exact equality. Diff the header cards between the two files to document byte-level differences such as added HISTORY cards, stripped mandatory-card comments, and normalized unit strings. Write the verdict (data values equal, headers not byte-identical) to a comparison report; the round trip is accepted only if every data-unit value matches and header changes are formatting-only consistent with re-serialization rather than data loss.

## Related tools

- **fits2hdf** (Converts the original FITS file into the HDFITS (HDF5) format as the forward leg of the round trip) — https://github.com/telegraphic/fits2hdf
- **hdf2fits** (Converts the HDFITS file back into FITS to produce the round-tripped file for comparison) — https://github.com/telegraphic/fits2hdf
- **Python** (Hosts astropy.io.fits / fits2hdf.pyhdfits loading and numpy element-wise data comparison plus header diffing)
- **HDF5** (Serialization backend of the HDFITS intermediate format used in the round trip)

## Examples

```
hdf2fits input_dir output_dir
```

## Evaluation signals

- Every data-unit array value in the round-tripped FITS file equals the corresponding value in the original (numpy element-wise equality)
- Header diff shows only formatting-level changes (added HISTORY cards, stripped mandatory-card comments, normalized unit strings) attributable to hdf2fits re-serialization
- The written report explicitly records 'data equal, headers not byte-identical' rather than claiming byte-level losslessness
- The round-tripped file is named and emitted with the same options as the forward fits2hdf conversion, so the comparison is apples-to-apples

## Limitations

- fits2hdf round-tripping is not byte-lossless, so the tool should not be used for archiving without careful comparison of input and output data
- HDFITS lacks better support for columns of data with masked values, which can affect round-trip fidelity for tables
- HDFITS lacks a specification for addressing uncertainties

## Evidence

- [other] fits2hdf round-tripping is not byte-lossless, so the tool cautions it should not be used for archiving: [evidence span withheld — anchor status: not_found]
- [methods] hdf2fits converts back into FITS: "To convert back into FITS, run ``hdf2fits``, which uses similar options::"
- [abstract] HDFITS presented transparently as FITS equivalent in Python: "by only changing the import lines in Python-based FITS utilities, HDFITS formatted data can be presented transparently as an in-memory FITS equivalent"
- [readme] fits2hdf is a FITS to HDF5 conversion utility: "`fits2hdf` is a conversion to utility to port FITS files to Hierarchical Data Format (HDF5)
files in the HDFITS format."
