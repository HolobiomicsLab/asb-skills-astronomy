---
name: hds-to-hdf5-data-model-mapping
description: Use when reconstructing how an HDS structure (structures, primitives,
  _CHAR*24 strings, _INTEGER, _LOGICAL, arrays of structures) is represented in an
  HDF5 file, or when building/validating an HDS-equivalent .h5 hierarchy with h5dump.
when_to_use_negative:
- When migrating existing HDS v4 (classic) files — the README states 'I'm not yet
  worrying about migration of HDS v4 files to HDF5'
- When the target structure requires slices or vectorization of a structure array
  — slicing/vectorizing breaks the HDSCELL collection-group convention
- When HDF5-native arrays of structures are assumed to exist — HDF5 does not support
  them; the explicit collection-group workaround must be used instead
tools:
- name: HDF5
  role: 'Underlying storage layer: HDS structures map to HDF5 groups with TYPE attributes
    and primitives to datasets; h5dump is used to inspect and validate the resulting
    file'
- name: HDS
  role: Source data model being re-implemented; the HDS API (datMap, datPut, hdsTrace,
    datCell, etc.) is layered on top of HDF5
  repo: https://github.com/Starlink
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2015.02.003
    title: Reimplementing the Hierarchical Data System using HDF5
  source_content_hash: sha256:e47edb9f954b72e769a1bc4a62f1d9834c54770e80f255ea10b490bea8e044bb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - HDF5
  - HDS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# hds-to-hdf5-data-model-mapping

## Summary

Maps the HDS (Starlink Hierarchical Data System) data model onto HDF5 constructs as implemented in the hds-v5 library: HDS structures become HDF5 groups with CLASS-style TYPE attributes, primitives become datasets, and arrays of structures become collection groups of HDSCELL children.

## When to use

Use when reconstructing how an HDS structure (structures, primitives, _CHAR*24 strings, _INTEGER, _LOGICAL, arrays of structures) is represented in an HDF5 file, or when building/validating an HDS-equivalent .h5 hierarchy with h5dump.

## When NOT to use

- When migrating existing HDS v4 (classic) files — the README states 'I'm not yet worrying about migration of HDS v4 files to HDF5'
- When the target structure requires slices or vectorization of a structure array — slicing/vectorizing breaks the HDSCELL collection-group convention
- When HDF5-native arrays of structures are assumed to exist — HDF5 does not support them; the explicit collection-group workaround must be used instead

## Inputs

- HDS data model specification or HDS file trace (e.g. HISTORY structure with CREATED _CHAR*24, CURRENT_RECORD _INTEGER, RECORDS(10) array of HIST_REC structures with DATE _CHAR*24 members)
- hds-v5 library (libhdsh5) mapping rules for TYPE/ISARRAY attributes and HDSCELL group naming

## Outputs

- HDF5 file whose group/dataset hierarchy mirrors the HDS structure (e.g. /HISTORY with TYPE=HISTORY, /RECORDS collection group with HDSCELL(1)..HDSCELL(10))
- h5dump_listing.txt — the h5dump text listing used as the structural validation artifact

## How to apply

Build an HDF5 file whose hierarchy mirrors the HDS structure: a top-level group (e.g. /HISTORY) carries a TYPE=HISTORY attribute, primitive components (/CREATED as _CHAR*24, /CURRENT_RECORD as _INTEGER) become datasets beneath it. Because HDF5 does not support arrays of structures, an HDS structure array such as RECORDS(10) <HIST_REC> becomes a collection group with TYPE=HIST_REC and ISARRAY=1 attributes holding child groups HDSCELL(1)..HDSCELL(10), each with its own TYPE attribute and primitive members (/DATE). When creating multi-dimensional primitives, transpose HDS Fortran-order dimensions into HDF5 C order when setting dataspace sizes, and store _LOGICAL as a 1-byte bitfield internally rather than a 4-byte int. Validate by running h5dump and checking that the listing structurally matches the expected mapping (groups, TYPE/ISARRAY attributes, HDSCELL children, and primitive members present).

## Related tools

- **HDF5** (Underlying storage layer: HDS structures map to HDF5 groups with TYPE attributes and primitives to datasets; h5dump is used to inspect and validate the resulting file)
- **HDS** (Source data model being re-implemented; the HDS API (datMap, datPut, hdsTrace, datCell, etc.) is layered on top of HDF5) — https://github.com/Starlink

## Examples

```
h5dump history.h5 > h5dump_listing.txt
```

## Evaluation signals

- h5dump listing shows /HISTORY group with Attr:TYPE=HISTORY and children /CREATED and /CURRENT_RECORD
- /RECORDS group carries TYPE=HIST_REC and ISARRAY=1 attributes and contains HDSCELL(1)..HDSCELL(10), each with TYPE=HIST_REC and a /DATE member
- Multi-dimensional primitive dataspaces reflect Fortran-to-C dimension transposition when compared against the HDS shape
- _LOGICAL values occupy 1 byte (bitfield) on disk rather than 4 bytes
- hdsTrace output converts /HDSCELL group names to HDS syntax, e.g. .HISTORY.RECORDS(5), by removing all occurrences of "/HDSCELL"

## Limitations

- HDF5 does not support arrays of structures, so slices of a structure array and vectorization (heavily used by HDSTRACE) break the HDSCELL collection-group convention
- Migration of HDS v4 files to HDF5 is explicitly not yet addressed
- datMap memory mapping is unsupported by HDF5; datUnmap must copy data back via anonymous mmap, adding failure modes if the program ends before datUnmap
- In the final system HDSCELL is replaced with a longer string longer than DAT__SZNAM to avoid accidental name collisions
- Type conversion to/from strings for numeric and logical types must happen in datPut before H5Dwrite, since HDF5 cannot do it in a single call

## Evidence

- [methods] Structure typing motivation: [evidence span withheld — anchor status: not_found]
