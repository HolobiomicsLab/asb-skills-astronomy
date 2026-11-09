---
name: starlink-test-dataset-generation
description: Use when reproducing the Section 7.1 benchmark datasets of the HDS/HDF5
  reimplementation paper, or when concrete HDSv4, gzipped-HDSv4, and HDF5-backend
  HDSv5 test files are needed for format-comparison experiments.
when_to_use_negative:
- The goal is converting or migrating existing HDSv4 files to HDF5 rather than generating
  fresh benchmark files — the README explicitly defers migration ('I'm not yet worrying
  about migration of HDS v4 files to HDF5').
- Benchmarking raw HDF5 write/read throughput independent of the HDS API layer — the
  HDSv5 library inserts HDS semantics (Fortran dimension order, EMS error wrapping,
  HDSCELL emulation) that dominate the results.
- The target format already supports arrays of structures natively (unlike HDF5),
  so the structural-emulation overhead being measured is absent.
edam_operation: http://edamontology.org/operation_2409
tools:
- name: HDSv5 (libhdsh5)
  role: Writes the HDF5-backend variant of each test dataset via the reimplemented
    HDS API
  repo: https://github.com/Starlink
- name: HDS
  role: Writes the native HDSv4 variants of the four benchmark datasets
  repo: https://github.com/Starlink
- name: NDF
  role: Data model underlying the kappa logo test dataset
- name: ADAM
  role: Parameter system whose ccdbig parameter file is one of the four test datasets;
    relies on string-to-type conversion in datPut
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.02.003
    title: Reimplementing the Hierarchical Data System using HDF5
  source_content_hash: sha256:ecd92c18bff1b5afcb26ba0978712847453852309b19377b78264e4eed9e2fad
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ADAM
  - HDS
  - HDSv5 (libhdsh5)
  - NDF
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# starlink-test-dataset-generation

## Summary

Generate the four HDS benchmark datasets used in the HDSv4 vs HDSv5 file-size and performance comparison (AGI graphics database, ADAM parameter file, SCUBA-2 acquisition file, kappa logo NDF) so each exists as a concrete HDS/NDF file on disk in multiple storage variants.

## When to use

Use when reproducing the Section 7.1 benchmark datasets of the HDS/HDF5 reimplementation paper, or when concrete HDSv4, gzipped-HDSv4, and HDF5-backend HDSv5 test files are needed for format-comparison experiments.

## When NOT to use

- The goal is converting or migrating existing HDSv4 files to HDF5 rather than generating fresh benchmark files — the README explicitly defers migration ('I'm not yet worrying about migration of HDS v4 files to HDF5').
- Benchmarking raw HDF5 write/read throughput independent of the HDS API layer — the HDSv5 library inserts HDS semantics (Fortran dimension order, EMS error wrapping, HDSCELL emulation) that dominate the results.
- The target format already supports arrays of structures natively (unlike HDF5), so the structural-emulation overhead being measured is absent.

## Inputs

- Starlink/hds-v5 source repository and its bundled test-data generator scripts (SpecDRE demo script, ccdbig, SCUBA-2 acquisition generator, kappa logo NDF)
- Working HDSv4 library for writing native HDS files
- HDSv5 (libhdsh5) library with HDF5 backend for writing HDF5-backed HDS files

## Outputs

- Four concrete HDS/NDF data files on disk (AGI graphics database, ADAM parameter file, SCUBA-2 acquisition file, kappa logo NDF)
- HDSv4 variants, gzipped HDSv4 variants, and HDF5-native-compression HDSv5 variants of each dataset
- table2_file_sizes.csv: 16 rows, columns for dataset name and file size in bytes

## How to apply

Clone the Starlink/hds-v5 repository and use the test-data generator scripts it provides to create the four datasets: the AGI graphics database (SpecDRE demo script, exercising arrays of structures and resizing), the ccdbig ADAM parameter file, the SCUBA-2 acquisition file (rich in table structures, written in a single operation), and the kappa logo NDF. Write each dataset natively in HDSv4 and record its byte size, then gzip each HDSv4 file to obtain the compressed variant. Write each dataset through the HDSv5 library (the libhdsh5 HDF5-backend implementation) with HDF5 native compression filters enabled, noting that the HDSv5 layer chunk datasets based on the size of the array being created. Assemble one row per dataset per variant (4 datasets x 4 variants = 16 rows) with dataset name and byte size into table2_file_sizes.csv. Rationale: the AGI dataset is the stress case because HDF5 has no native array-of-structures support, so HDSv5 must emulate it with explicit HDSCELL collection groups, inflating file size by more than 5x with 20% empty space.

## Related tools

- **HDSv5 (libhdsh5)** (Writes the HDF5-backend variant of each test dataset via the reimplemented HDS API) — https://github.com/Starlink
- **HDS** (Writes the native HDSv4 variants of the four benchmark datasets) — https://github.com/Starlink
- **NDF** (Data model underlying the kappa logo test dataset)
- **ADAM** (Parameter system whose ccdbig parameter file is one of the four test datasets; relies on string-to-type conversion in datPut)

## Examples

```
git clone https://github.com/Starlink/hds-v5 && cd hds-v5
```

## Evaluation signals

- table2_file_sizes.csv exists and contains exactly 16 rows covering all 4 datasets x 4 storage variants
- Recorded byte sizes match the paper's Section 7 comparison, including the AGI graphics database result where the HDF5 variant is more than five times larger than HDSv4
- For the AGI file, the empty-space overhead (20% of the HDF5 file size) is reproducible from the recorded sizes
- Each generated file is readable back through its respective library (HDSv4 or HDSv5) without error
- HDSv5 output files are directly readable by general-purpose HDF5 tools such as h5py

## Limitations

- HDF5 does not support arrays of structures, so HDSv5 emulates them via explicit HDSCELL collection groups; slices and vectorization of array structures break this emulation (relevant to HDSTRACE)
- HDSv5 chunking is heuristic — it chunks based on the size of the array being created with H5S_UNLIMITED as the maximum size — which can inflate file size unpredictably
- A _LOGICAL is stored as a 1-byte bitfield in HDSv5 versus a 4-byte int in HDSv4, so logical-heavy datasets will not size-compare identically
- Migration of HDSv4 files to HDF5 is not addressed by the current library

## Evidence

- [results] AGI graphics database as the structural stress case: [evidence span withheld — anchor status: not_found]
- [intro] SCUBA-2 acquisition file as a test dataset: [evidence span withheld — anchor status: not_found]
- [discussion] Obtaining the source repository: [evidence span withheld — anchor status: not_found]
