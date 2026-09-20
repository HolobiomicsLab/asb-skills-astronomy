---
name: fits-to-hdf5-data-model-mapping
description: Use when converting FITS files (or MeasurementSets) into HDF5 while preserving
  the FITS data model of HDUs, header cards, image datasets, and binary-table columns.
when_to_use_negative:
- The target software stack only supports FITS — HDF5 is not well supported by data
  reduction software packages and image viewers; use this skill only when downstream
  tools can read HDF5 or you plan round-trip conversion back to FITS.
- You need features with no HDFITS equivalent yet, such as the FITS subtractive dithering
  technique, a specification for uncertainties, or columns of data with masked values.
- The data are arbitrary Python objects rather than FITS/MS scientific data — use
  hickle (HDF5-based pickle replacement) instead.
tools:
- name: fits2hdf
  role: 'Core conversion utility: maps FITS/MS/HDF data into the IDI interchange format
    and serializes to HDFITS via read_fits/export_hdf (and hdf2fits for the reverse
    direction)'
  repo: https://github.com/telegraphic/fits2hdf
- name: h5py
  role: Pythonic HDF5 interface on which fits2hdf's hdfio export/read functions are
    built; also used to validate the output file
- name: Astropy
  role: Community Python package for Astronomy used by fits2hdf for FITS handling;
    astropy.io.fits is the baseline that pyhdfits transparently replaces
- name: bitshuffle
  role: Optional HDF5 filter (number 32008) for bit-level rearrangement improving
    compression of typed binary data, used for radio astronomy datasets in the LEDA
    experiment
  repo: https://github.com/kiyo-masui/bitshuffle
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2015.05.001
    title: 'HDFITS: Porting the FITS data model to HDF5'
  source_content_hash: sha256:718ffabc9baf459fb5f097a27b9d701eaad4535b513b582e0974e0a7a2532119
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/telegraphic/fits2hdf
  tools:
  - Astropy
  - Python
  - bitshuffle
  - fits2hdf
  - h5py
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# fits-to-hdf5-data-model-mapping

## Summary

Ports FITS files into the HDFITS HDF5 format using fits2hdf, preserving the FITS data model (HDUs, header cards as attributes, image and binary-table datasets) via CLASS attributes. Enables faster reads, better compression, and round-trip conversion back to FITS.

## When to use

Use when converting FITS files (or MeasurementSets) into HDF5 while preserving the FITS data model of HDUs, header cards, image datasets, and binary-table columns.

## When NOT to use

- The target software stack only supports FITS — HDF5 is not well supported by data reduction software packages and image viewers; use this skill only when downstream tools can read HDF5 or you plan round-trip conversion back to FITS.
- You need features with no HDFITS equivalent yet, such as the FITS subtractive dithering technique, a specification for uncertainties, or columns of data with masked values.
- The data are arbitrary Python objects rather than FITS/MS scientific data — use hickle (HDF5-based pickle replacement) instead.

## Inputs

- FITS file (e.g. an SDSS FITS file) with HDUs containing headers, image data, and/or binary tables
- MeasurementSet (MS) data (fits2hdf also supports MS-to-HDF5 conversion)

## Outputs

- HDFITS-format HDF5 file (e.g. sdss_hdfits.h5) with root group CLASS=HDFITS, HDU groups CLASS=HDU, datasets CLASS=IMAGE / CLASS=COLUMN, and parent groups CLASS=DATA_GROUP
- FITS header cards preserved as HDF5 group attributes (key-value metadata)

## How to apply

Use the fits2hdf utility, which first maps data from FITS/MS/HDF into an in-memory interchange format (IDI). Read the FITS file with read_fits from fits2hdf.io.fitsio, then serialize with export_hdf from fits2hdf.io.hdfio (built on h5py): each HDU becomes a root-level HDF5 group carrying a CLASS=HDU attribute, with FITS header cards stored as group attributes (key-value metadata). Image payloads are written as datasets annotated CLASS=IMAGE and binary-table columns as datasets annotated CLASS=COLUMN inside a parent group annotated CLASS=DATA_GROUP; the root group carries CLASS=HDFITS to identify the file. The CLASS keyword is the sole mechanism for identifying HDUs, images, and tables within the HDF5 hierarchy, so these attributes must be set exactly. Optionally apply compression (e.g. bitshuffle, HDF5 filter 32008, suited to radio astronomy data); to go back to FITS, run hdf2fits with similar options.

## Related tools

- **fits2hdf** (Core conversion utility: maps FITS/MS/HDF data into the IDI interchange format and serializes to HDFITS via read_fits/export_hdf (and hdf2fits for the reverse direction)) — https://github.com/telegraphic/fits2hdf
- **h5py** (Pythonic HDF5 interface on which fits2hdf's hdfio export/read functions are built; also used to validate the output file)
- **Astropy** (Community Python package for Astronomy used by fits2hdf for FITS handling; astropy.io.fits is the baseline that pyhdfits transparently replaces)
- **bitshuffle** (Optional HDF5 filter (number 32008) for bit-level rearrangement improving compression of typed binary data, used for radio astronomy datasets in the LEDA experiment) — https://github.com/kiyo-masui/bitshuffle

## Examples

```
from fits2hdf.io.fitsio import read_fits; from fits2hdf.io.hdfio import export_hdf; idi = read_fits('sdss.fits'); export_hdf(idi, 'sdss_hdfits.h5')
```

## Evaluation signals

- Output file (e.g. sdss_hdfits.h5) exists and is a valid HDF5 file readable by h5py.
- Root group of the output carries the CLASS=HDFITS attribute.
- Each HDU appears as a root-level HDF5 group with CLASS=HDU, and FITS header cards are present as group attributes.
- Image payloads are datasets with CLASS=IMAGE; binary-table columns are datasets with CLASS=COLUMN nested under groups with CLASS=DATA_GROUP.
- Round-trip conversion with hdf2fits produces a FITS file consistent with the original (backwards-compatibility with FITS maintained).

## Limitations

- HDF5 lacks widespread adoption in astronomy because it is not well supported by data reduction software packages and image viewers.
- HDFITS lacks a specification for addressing uncertainties and needs better support for columns of data with masked values.
- There is currently no HDF5/HDFITS equivalent to the FITS subtractive dithering technique.
- Extending the HDFITS data model requires community input and discussion that has not yet occurred; lossy compression algorithms were not compared.

## Evidence
