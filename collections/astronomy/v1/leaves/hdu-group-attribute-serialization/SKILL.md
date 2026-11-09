---
name: hdu-group-attribute-serialization
description: Use when converting FITS files (or MeasurementSets) into the HDFITS HDF5
  format and the FITS hierarchy of HDUs, headers, images, and binary-table columns
  must be preserved as groups, datasets, and attributes.
when_to_use_negative:
- The target is a generic Python object dump rather than a FITS-derived data model
  — use a tool like hickle instead, which serializes arbitrary Python objects to HDF5
- The output must remain a legacy FITS file for existing FITS-only reduction software
  or image viewers, since HDF5 is not well supported by such packages
- Only round-tripping to the Starlink NDF format is needed — the separate fits2ndf
  utility stores header cards as-is (as strings) rather than the HDFITS CLASS-attribute
  scheme
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: fits2hdf
  role: Provides read_fits/export_hdf and the HDFITS CLASS-attribute convention for
    mapping FITS HDUs, headers, images, and table columns onto HDF5 groups, datasets,
    and attributes
  repo: https://github.com/telegraphic/fits2hdf
- name: h5py
  role: Pythonic HDF5 interface underlying fits2hdf's export_hdf serialization of
    HDU groups and CLASS attributes
- name: Astropy
  role: Parses the source FITS file (astropy.io.fits) whose HDUs and header cards
    are mapped into HDF5; fits2hdf's pyhdfits module can substitute for astropy.io.fits
    on import
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2015.05.001
    title: 'HDFITS: Porting the FITS data model to HDF5'
  source_content_hash: sha256:939eb78596c2624caf4ee6336220cd90b470a371a3485ef85bc2411389825d7f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/telegraphic/fits2hdf
  tools:
  - Astropy
  - Python
  - fits2hdf
  - h5py
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# hdu-group-attribute-serialization

## Summary

Serializes the FITS data model into HDF5 by writing each header data unit as a root-level HDF5 group with a CLASS=HDU attribute and encoding header cards, image payloads, and table columns as CLASS-annotated attributes, as defined by the HDFITS format and implemented in fits2hdf.

## When to use

Use when converting FITS files (or MeasurementSets) into the HDFITS HDF5 format and the FITS hierarchy of HDUs, headers, images, and binary-table columns must be preserved as groups, datasets, and attributes.

## When NOT to use

- The target is a generic Python object dump rather than a FITS-derived data model — use a tool like hickle instead, which serializes arbitrary Python objects to HDF5
- The output must remain a legacy FITS file for existing FITS-only reduction software or image viewers, since HDF5 is not well supported by such packages
- Only round-tripping to the Starlink NDF format is needed — the separate fits2ndf utility stores header cards as-is (as strings) rather than the HDFITS CLASS-attribute scheme

## Inputs

- FITS file (e.g. SDSS FITS file) readable by Astropy
- MeasurementSet (MS) for the MS-to-HDF5 path
- in-memory IDI interchange object produced by read_fits / read_ms

## Outputs

- HDF5 file in HDFITS format (e.g. sdss_hdfits.h5) with root group attribute CLASS=HDFITS
- root-level HDU groups carrying CLASS=HDU with FITS header cards stored as group attributes
- datasets annotated CLASS=IMAGE and CLASS=COLUMN nested under groups annotated CLASS=DATA_GROUP

## How to apply

Map data from FITS/MS/HDF into the in-memory IDI interchange format first (e.g. read_fits from fits2hdf.io.fitsio), since fits2hdf works by staging data in this representation before serialization. Serialize each HDU as a root-level HDF5 group carrying a CLASS=HDU attribute, writing the FITS header cards into the group attributes as key-value metadata. Write image payloads as datasets annotated CLASS=IMAGE and binary-table columns as datasets annotated CLASS=COLUMN inside a parent group annotated CLASS=DATA_GROUP, preserving the FITS data model hierarchy. Mark the root group with CLASS=HDFITS so the file is identifiable as a HDFITS file, and export with export_hdf from fits2hdf.io.hdfio (built on h5py). The keyword CLASS is the single attribute used to distinguish HDUs, image vs table datasets, and the file itself.

## Related tools

- **fits2hdf** (Provides read_fits/export_hdf and the HDFITS CLASS-attribute convention for mapping FITS HDUs, headers, images, and table columns onto HDF5 groups, datasets, and attributes) — https://github.com/telegraphic/fits2hdf
- **h5py** (Pythonic HDF5 interface underlying fits2hdf's export_hdf serialization of HDU groups and CLASS attributes)
- **Astropy** (Parses the source FITS file (astropy.io.fits) whose HDUs and header cards are mapped into HDF5; fits2hdf's pyhdfits module can substitute for astropy.io.fits on import)

## Examples

```
from fits2hdf.io.fitsio import read_fits; from fits2hdf.io.hdfio import export_hdf; idi = read_fits('sdss.fits'); export_hdf(idi, 'sdss_hdfits.h5')
```

## Evaluation signals

- sdss_hdfits.h5 exists and is a valid HDF5 file readable by h5py
- Root group of the output carries the CLASS=HDFITS attribute
- Every serialized header data unit appears as a root-level HDF5 group with a CLASS=HDU attribute
- Image and table payloads carry CLASS=IMAGE / CLASS=COLUMN attributes within parent groups annotated CLASS=DATA_GROUP
- FITS header cards are present as group attributes (key-value metadata) matching the source HDU headers

## Limitations

- HDF5 is not well supported by data reduction software packages and image viewers, limiting downstream consumption of the serialized file
- HDFITS lacks a specification for addressing uncertainties and needs better support for columns of data with masked values
- Extending the HDFITS data model requires community input and discussion that has not yet occurred; the HDF5 API provides only a low-level approach to loading data

## Evidence

- [methods] HDU identification via CLASS attribute: [evidence span withheld — anchor status: not_found]
