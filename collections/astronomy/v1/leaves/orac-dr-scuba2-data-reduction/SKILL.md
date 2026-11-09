---
name: orac-dr-scuba2-data-reduction
description: Use when benchmarking the HDF5-backed HDSv5 library against native HDSv4
  on a real-world reduction pipeline — specifically timing the SCUBA-2 orac-dr reduction
  of an acquisition file that writes bulk data plus table structures in a single operation.
when_to_use_negative:
- The target workload is many small files with heavy I/O (the ccdpack/ccdbig regime),
  where HDSv4 is much faster and micro-benchmarks are the appropriate test.
- The goal is migrating existing HDS v4 files to HDF5 — the hds-v5 library explicitly
  does not yet address that migration.
- Reading SCUBA-2 HDF5 output without the Starlink/HDS stack (e.g. with h5py) — this
  skill requires builds of the Starlink software against both HDS libraries.
tools:
- name: HDS (HDSv4)
  role: Native Starlink Hierarchical Data System build providing the baseline (v4)
    wall-clock timing for the orac-dr reduction
  repo: https://github.com/Starlink
- name: hds-v5
  role: HDF5-backed reimplementation of the HDS API; the Starlink stack is rebuilt
    against it to record the v5 timing
  repo: https://github.com/Starlink/hds-v5
- name: HDF5
  role: Storage layer beneath hds-v5; its lack of native memory mapping forces the
    datMap mmap/datUnmap copy-back path timed by the benchmark
- name: NDF
  role: Extensible N-Dimensional Data Format whose HDS-backed files the orac-dr reduction
    reads and writes
- name: orac-dr
  role: SCUBA-2 data reduction pipeline exercised on the acquisition file as the real-world
    benchmark task under both builds
  repo: https://github.com/Starlink
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.02.003
    title: Reimplementing the Hierarchical Data System using HDF5
  source_content_hash: sha256:f608a58404cb4c29a60a2f34be6269bfa137383542b0c462cf60008b76cf2df2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - HDF5
  - HDS
  - HDS (HDSv4)
  - NDF
  - hds-v5
  - orac-dr
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# orac-dr-scuba2-data-reduction

## Summary

Runs the SCUBA-2 orac-dr data reduction as the real-world performance benchmark comparing the HDF5-backed HDSv5 library with native HDSv4, timing the reduction of a SCUBA-2 acquisition file (Bintley et al., 2014) that contains bulk data plus table structures written in a single operation.

## When to use

Use when benchmarking the HDF5-backed HDSv5 library against native HDSv4 on a real-world reduction pipeline — specifically timing the SCUBA-2 orac-dr reduction of an acquisition file that writes bulk data plus table structures in a single operation.

## When NOT to use

- The target workload is many small files with heavy I/O (the ccdpack/ccdbig regime), where HDSv4 is much faster and micro-benchmarks are the appropriate test.
- The goal is migrating existing HDS v4 files to HDF5 — the hds-v5 library explicitly does not yet address that migration.
- Reading SCUBA-2 HDF5 output without the Starlink/HDS stack (e.g. with h5py) — this skill requires builds of the Starlink software against both HDS libraries.

## Inputs

- SCUBA-2 acquisition file (Bintley et al., 2014) containing bulk data plus table structures written in a single operation
- Starlink software build linked against native HDS (HDSv4)
- Starlink software build linked against the hds-v5 (HDF5-backed) library

## Outputs

- Wall-clock timings (seconds) of the orac-dr SCUBA-2 reduction under both the HDSv4 and HDSv5 builds
- v5/v4 timing ratio for the orac-dr row of the Table 3 comparison, assembled in table3_timings.csv

## How to apply

Build the Starlink software stack from github.com/Starlink twice: once linked against the native HDS (HDSv4) and once against the hds-v5 (HDF5-backed) library, then run the orac-dr SCUBA-2 data reduction on the SCUBA-2 acquisition file under both builds and record wall-clock times in seconds. The acquisition file is chosen because it contains lots of data as well as table structures written in a single operation, so it exercises HDSv5's emulation of arrays of structures (HDSCELL collection groups) and single-operation bulk writes through datPut. Compute the v5/v4 timing ratio for the orac-dr row of the Table 3 comparison and write the per-benchmark seconds and ratios to table3_timings.csv. Where the reduction exercises datMap-based memory mapping, repeat the run with mapping disabled because HDSv5's datMap must mmap an anonymous memory area and datUnmap copies the data back rather than using native HDF5 mapping. Judge success by a v5/v4 ratio within a few per cent of 1.0, matching the reported finding for real-world tasks with reasonably-sized datasets; do not generalize from the small-file, I/O-heavy benchmarks (ccdpack, ccdbig) where HDSv4 is much faster.

## Related tools

- **HDS (HDSv4)** (Native Starlink Hierarchical Data System build providing the baseline (v4) wall-clock timing for the orac-dr reduction) — https://github.com/Starlink
- **hds-v5** (HDF5-backed reimplementation of the HDS API; the Starlink stack is rebuilt against it to record the v5 timing) — https://github.com/Starlink/hds-v5
- **HDF5** (Storage layer beneath hds-v5; its lack of native memory mapping forces the datMap mmap/datUnmap copy-back path timed by the benchmark)
- **NDF** (Extensible N-Dimensional Data Format whose HDS-backed files the orac-dr reduction reads and writes)
- **orac-dr** (SCUBA-2 data reduction pipeline exercised on the acquisition file as the real-world benchmark task under both builds) — https://github.com/Starlink

## Evaluation signals

- Wall-clock times in seconds are recorded for the same SCUBA-2 acquisition-file reduction under both the HDSv4 and hds-v5 builds, enabling a direct v5/v4 ratio.
- The orac-dr v5/v4 ratio should be within a few per cent of 1.0, consistent with the reported finding for real-world tasks with larger datasets.
- The run must exercise the single-operation write of bulk data plus table structures, verifying that HDSv5's HDSCELL collection-group emulation of arrays of structures does not break the reduction.
- Rows that exercise datMap are re-run with memory mapping disabled to separate the mmap-anonymous-memory plus datUnmap copy-back overhead from core I/O performance.
- Per-benchmark seconds and v5/v4 ratios land in table3_timings.csv with the orac-dr row matching the Table 3 structure.

## Limitations

- HDF5 does not support native memory mapping, so HDSv5's datMap must mmap an anonymous memory area and datUnmap copies the data back to the underlying dataset, adding overhead and removing const from the API.
- Migration of existing HDS v4 files to HDF5 is not yet addressed by hds-v5.
- HDF5 does not support arrays of structures; structure cells are emulated as HDSCELL collection groups, and slicing/vectorization of structure arrays (used heavily by HDSTRACE) require extra work.
- A few of the approximately 150 HDS API functions — those querying low-level implementation details — are not implemented in hds-v5.
- Small-file, I/O-heavy tasks run much slower under HDSv5, so this real-world orac-dr result must not be generalized to that regime.

## Evidence

- [intro] Choice of SCUBA-2 acquisition file as benchmark input: [evidence span withheld — anchor status: not_found]
- [discussion] HDSv5 performance parity finding: [evidence span withheld — anchor status: not_found]
