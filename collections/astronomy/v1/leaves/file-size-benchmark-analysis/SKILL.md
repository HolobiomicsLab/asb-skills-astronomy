---
name: file-size-benchmark-analysis
description: Use when a paper or migration plan compares file sizes of an old vs new
  data format (e.g. HDSv4 vs HDSv5/HDF5) for a fixed set of benchmark datasets and
  you need to reproduce or extend the byte-size comparison table.
when_to_use_negative:
- The goal is performance/runtime benchmarking of API calls rather than on-disk storage
  comparison — use timing benchmarks instead
- The datasets are plain HDF5 files with no HDS structure semantics, where HDSCELL
  group expansion does not apply
- Migration of existing HDSv4 archives to HDF5 is the question — the hds-v5 author
  explicitly does not yet address HDS v4 file migration
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: HDS
  role: Legacy Hierarchical Data System library used to write the HDSv4 baseline files
  repo: https://github.com/Starlink
- name: HDF5
  role: Backend storage format on which the hds-v5 library writes the compressed variant
    files
- name: hds-v5
  role: Reimplementation of the HDS API on top of HDF5, used to produce the new-format
    benchmark files
  repo: https://github.com/Starlink/hds-v5
- name: NDF
  role: Extensible N-Dimensional Data Format whose data files (e.g. the kappa logo
    NDF) serve as one benchmark dataset
- name: ADAM
  role: Starlink parameter system whose ccdbig parameter file is one of the four benchmark
    datasets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.02.003
    title: Reimplementing the Hierarchical Data System using HDF5
  source_content_hash: sha256:4ee8e4dcd73f68a0aa531c9156e57e104d5971179ab3d608212b1f5dfd48f3ca
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ADAM
  - HDF5
  - HDS
  - NDF
  - hds-v5
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# file-size-benchmark-analysis

## Summary

Benchmarking method for comparing on-disk byte sizes of the same logical dataset written in competing storage formats (here HDSv4, gzipped HDSv4, and HDF5-backed HDSv5 with HDF5 native compression), used to quantify format-storage trade-offs before archive-wide migration decisions.

## When to use

Use when a paper or migration plan compares file sizes of an old vs new data format (e.g. HDSv4 vs HDSv5/HDF5) for a fixed set of benchmark datasets and you need to reproduce or extend the byte-size comparison table.

## When NOT to use

- The goal is performance/runtime benchmarking of API calls rather than on-disk storage comparison — use timing benchmarks instead
- The datasets are plain HDF5 files with no HDS structure semantics, where HDSCELL group expansion does not apply
- Migration of existing HDSv4 archives to HDF5 is the question — the hds-v5 author explicitly does not yet address HDS v4 file migration

## Inputs

- Four HDS/NDF benchmark datasets: AGI graphics database (SpecDRE demo), ADAM parameter file (ccdbig), SCUBA-2 acquisition file, kappa logo NDF
- Starlink/hds-v5 source repository with test-data generator scripts
- HDSv4 library and gzip for legacy-format writes
- hds-v5 library (HDF5 backend) with native compression filters enabled

## Outputs

- table2_file_sizes.csv with 16 rows (4 datasets x 4 variants) containing dataset name and file size in bytes
- Per-dataset size ratios (HDSv5/HDSv4) and identification of overhead sources (e.g. empty space fraction)

## How to apply

Generate each benchmark dataset as a concrete file on disk using the repository's test-data generators (SpecDRE demo script for the AGI graphics database, ccdbig for the ADAM parameter file, the SCUBA-2 acquisition file, and the kappa logo NDF). For each dataset, write a native HDSv4 file and a gzip-compressed variant of that HDSv4 file, recording exact byte sizes; then write each dataset through the hds-v5 library (HDF5 backend) with HDF5 native compression filters enabled, recording those sizes. Assemble a CSV with one row per dataset-variant combination (4 datasets x 4 variants = 16 rows) with columns for dataset name and size in bytes. Interpret results in light of format-inherent overhead: the AGI graphics database, which makes extensive use of arrays of structures and resizing, becomes more than five times larger in the HDF5 variant, with 20% of that accounted for by empty space, because HDF5 does not support arrays of structures and HDSv5 must materialize explicit HDSCELL groups.

## Related tools

- **HDS** (Legacy Hierarchical Data System library used to write the HDSv4 baseline files) — https://github.com/Starlink
- **HDF5** (Backend storage format on which the hds-v5 library writes the compressed variant files)
- **hds-v5** (Reimplementation of the HDS API on top of HDF5, used to produce the new-format benchmark files) — https://github.com/Starlink/hds-v5
- **NDF** (Extensible N-Dimensional Data Format whose data files (e.g. the kappa logo NDF) serve as one benchmark dataset)
- **ADAM** (Starlink parameter system whose ccdbig parameter file is one of the four benchmark datasets)

## Examples

```
python make_test_data.py --dataset agi_graphics && gzip -k agi_graphics.sdf && hds5_write --compress native --output agi_graphics.h5
```

## Evaluation signals

- table2_file_sizes.csv exists and contains exactly 16 rows covering all dataset-variant combinations
- Recorded byte sizes match the file-size comparison reported in the paper (e.g. AGI graphics database HDF5 variant more than five times larger than HDSv4)
- Empty-space fraction for the resized/array-of-structures dataset is consistent with the reported ~20%
- Each of the four datasets (AGI graphics database, ADAM parameter file, SCUBA-2 acquisition file, kappa logo NDF) is represented in all four storage variants
- Gzipped HDSv4 and HDF5-native-compression variants are distinguished as separate rows rather than collapsed

## Limitations

- HDF5 does not support arrays of structures, so HDSv5 must add explicit HDSCELL collection groups, inflating sizes for datasets that use arrays of structures and resizing (AGI graphics database)
- The HDSv5 library currently chunks based on the size of the array being created, in the absence of any HDS API guidance on chunking, which affects resulting file sizes
- Migration of HDS v4 files to HDF5 is not yet addressed by the hds-v5 implementation, so the benchmark does not measure conversion fidelity or cost
- Results are specific to the four test datasets in Section 7.1 and may not generalize to all HDS file patterns (e.g. the ~4 million HDS files in the JCMT Science Archive)

## Evidence

- [results] For the AGI graphics database, which makes extensive use of arrays of structures and resizing, the HDF5 variant is more than five times larger than the HDSv4 variant: [evidence span withheld — anchor status: not_found]
