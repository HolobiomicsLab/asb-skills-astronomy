---
name: hds-hdf5-file-format-conversion
description: Use when legacy HDS/HDSv4 astronomy data files (NDF images, ADAM parameter
  files, SCUBA-2 acquisition files) must be written or re-expressed as HDF5 via the
  HDSv5 drop-in library, or when comparing HDSv4 vs HDSv5 file sizes.
when_to_use_negative:
- Input is already HDF5 or a format with a standard FITS-to-HDF5 conversion model
  — the paper notes no agreed standard model for FITS to HDF5 conversion exists, so
  this HDS-specific path does not apply.
- The goal is migrating existing HDS v4 files into HDF5 — the hds-v5 author explicitly
  states 'I'm not yet worrying about migration of HDS v4 files to HDF5'; no converter
  or forwarding wrapper is provided yet.
- The workload relies on HDS API routines that query low-level implementation details
  — those handful of the ~150 HDS API functions were not implemented in the HDF5 prototype.
tools:
- name: HDS (Starlink Hierarchical Data System)
  role: Original file format and API whose ~150-function interface is being reproduced;
    HDSv4 writer used for the baseline file-size variant
- name: HDF5
  role: Backend storage format; HDS structures are mapped onto HDF5 groups/datasets
    with native compression filters and size-based chunking
- name: NDF (extensible N-Dimensional Data Format)
  role: Data model built on HDS that the conversion aims to make accessible to broader
    (h5py-capable) users; kappa logo NDF is one benchmark dataset
- name: ADAM
  role: Parameter system whose string-typed parameters rely on HDS type conversion
    (handled in datPut before H5Dwrite); ccdbig generates the ADAM parameter-file
    benchmark
- name: libhdsh5 (Starlink/hds-v5)
  role: Drop-in re-implementation of the HDS API on top of HDF5; wraps all HDF5 calls
    with EMS error-stack forwarding
  repo: https://github.com/Starlink
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.02.003
    title: Reimplementing the Hierarchical Data System using HDF5
  source_content_hash: sha256:bb27977e450042ebc4a1deff7b7e57d146706fddac95e5b934e91b1049edd19d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ADAM
  - HDF5
  - HDS
  - HDS (Starlink Hierarchical Data System)
  - NDF
  - NDF (extensible N-Dimensional Data Format)
  - libhdsh5 (Starlink/hds-v5)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# hds-hdf5-file-format-conversion

## Summary

Method for writing HDS (Starlink Hierarchical Data System) data structures through the libhdsh5 re-implementation so that HDSv4-era files (NDF, ADAM parameter files, array-of-structure containers) are stored as HDF5 files, and for benchmarking the resulting file sizes against HDSv4 and gzipped variants.

## When to use

Use when legacy HDS/HDSv4 astronomy data files (NDF images, ADAM parameter files, SCUBA-2 acquisition files) must be written or re-expressed as HDF5 via the HDSv5 drop-in library, or when comparing HDSv4 vs HDSv5 file sizes.

## When NOT to use

- Input is already HDF5 or a format with a standard FITS-to-HDF5 conversion model — the paper notes no agreed standard model for FITS to HDF5 conversion exists, so this HDS-specific path does not apply.
- The goal is migrating existing HDS v4 files into HDF5 — the hds-v5 author explicitly states 'I'm not yet worrying about migration of HDS v4 files to HDF5'; no converter or forwarding wrapper is provided yet.
- The workload relies on HDS API routines that query low-level implementation details — those handful of the ~150 HDS API functions were not implemented in the HDF5 prototype.

## Inputs

- HDSv4/HDS (Starlink Hierarchical Data System) files or generator scripts: SpecDRE demo script (AGI graphics database), ccdbig (ADAM parameter file), SCUBA-2 acquisition file, kappa logo NDF
- Starlink hds-v5 source repository (libhdsh5, the HDS API implemented on HDF5)
- HDF5 library with native compression filters

## Outputs

- HDF5 files written through the HDSv5 library (HDS/HDF5 backend) for each dataset
- table2_file_sizes.csv with one row per dataset per storage variant (HDSv4, gzipped HDSv4, HDSv5, HDSv5 with HDF5 native compression), 16 rows total
- HDS-syntax trace output (e.g. .HISTORY.RECORDS(5)) recovered from /HDSCELL group hierarchy via hdsTrace

## How to apply

Obtain the Starlink hds-v5 source from GitHub and generate or obtain the benchmark datasets (the AGI graphics database via the SpecDRE demo script, the ADAM parameter file via ccdbig, the SCUBA-2 acquisition file, and the kappa logo NDF). Write each dataset once in native HDSv4 format and once gzipped, recording byte sizes, then write each through the HDSv5 library (the HDF5-backed implementation of the ~150-function HDS API) with HDF5 native compression filters enabled, again recording byte sizes. Be aware of the structural translation: HDF5 does not support arrays of structures, so HDS structure arrays become explicit collection groups (e.g. /HISTORY/RECORDS/HDSCELL(5)) that hdsTrace maps back to HDS syntax like .HISTORY.RECORDS(5). Because HDF5 requires chunking when a maximum size is set (H5S_UNLIMITED is used to allow reshaping), the library chunks based on the size of the array being created, which affects compressed file size. Assemble the sizes as a per-dataset, per-variant CSV (4 datasets x 4 variants = 16 rows, e.g. table2_file_sizes.csv) and check them against the paper's reported comparison, noting the known worst case: the AGI graphics database is more than five times larger in HDSv5, with 20% of that from empty space.

## Related tools

- **HDS (Starlink Hierarchical Data System)** (Original file format and API whose ~150-function interface is being reproduced; HDSv4 writer used for the baseline file-size variant)
- **HDF5** (Backend storage format; HDS structures are mapped onto HDF5 groups/datasets with native compression filters and size-based chunking)
- **NDF (extensible N-Dimensional Data Format)** (Data model built on HDS that the conversion aims to make accessible to broader (h5py-capable) users; kappa logo NDF is one benchmark dataset)
- **ADAM** (Parameter system whose string-typed parameters rely on HDS type conversion (handled in datPut before H5Dwrite); ccdbig generates the ADAM parameter-file benchmark)
- **libhdsh5 (Starlink/hds-v5)** (Drop-in re-implementation of the HDS API on top of HDF5; wraps all HDF5 calls with EMS error-stack forwarding) — https://github.com/Starlink

## Examples

```
git clone https://github.com/Starlink/hds-v5
```

## Evaluation signals

- table2_file_sizes.csv exists and contains exactly 16 rows covering all 4 dataset x 4 storage-variant combinations with byte sizes matching the paper's Section 7.1 comparison
- hdsTrace on a converted array-of-structures container returns HDS syntax (e.g. .HISTORY.RECORDS(5)) with all /HDSCELL occurrences removed
- All but a handful of the ~150 HDS API functions succeed; only routines querying low-level implementation details are expected to be unavailable
- Growth relative to HDSv4 is bounded for simple files, with the known exception that the AGI graphics database (heavy arrays of structures and resizing) is more than five times larger, ~20% of it empty space
- Type-conversion-dependent paths (ADAM parameter files, where parameters are supplied as strings) round-trip numeric/logical values correctly through datPut before H5Dwrite

## Limitations

- HDF5 does not support arrays of structures; they are emulated with explicit HDSCELL collection groups, and slices or vectorization of structure arrays (as used by HDSTRACE) require extra work and can break this mapping
- Memory mapping (datMap/datUnmap) is not supported by HDF5, so an anonymous mmap area is used and data copied back on datUnmap, with risks if the program ends before datUnmap or if a primitive is mapped twice with different locators
- _LOGICAL is stored as a 1-byte bitfield internally instead of HDS's 4-byte int, and hdsdim is a 32-bit int versus HDF5's 64-bit hsize_t, so large-dimension or tightly-typed data may not round-trip identically
- datMap/datUnmap must lose the const in the API; hdsLock cannot access the underlying file descriptor; datRef/datMsg do not yet report the full file path and datParen does not detect the top of the group hierarchy
- Chunking is chosen automatically based on array size (H5S_UNLIMITED maximum) because the HDS API offers no guidance, which can inflate file size (empty space in the AGI case)

## Evidence

- [discussion] All but a handful of the approximately 150 HDS API functions have been implemented, with the remaining few being the routines that query low level implementation details.: [evidence span withheld — anchor status: not_found]
