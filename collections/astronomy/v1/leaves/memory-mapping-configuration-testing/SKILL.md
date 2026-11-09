---
name: memory-mapping-configuration-testing
description: Use when porting or profiling HDS datMap/datUnmap semantics onto HDF5,
  or when benchmark timing ratios (v5/v4) are unexpectedly high for map-heavy routines
  and you need to isolate the mmap/copy-back overhead.
when_to_use_negative:
- When the file being opened is a native HDS v4 file — the hds-v5 library does not
  handle HDS v4 files and migration is explicitly out of scope
- When the analysis target is HDF5-native mapping features (compression-filter-aware
  mapping) — HDSv5's datMap cannot use them and only emulates mapping via anonymous
  mmap
tools:
- name: HDS (HDSv5 on HDF5)
  role: 'Library under test: datMap/datUnmap emulate memory mapping over HDF5 datasets;
    exercised via the Starlink benchmarks'
  repo: https://github.com/Starlink
- name: HDF5
  role: Underlying storage layer whose lack of native memory mapping forces the anonymous-mmap
    and copy-back implementation
- name: NDF
  role: Starlink N-dimensional data format whose applications run on top of HDS and
    exercise the mapping path in benchmarks
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.02.003
    title: Reimplementing the Hierarchical Data System using HDF5
  source_content_hash: sha256:ba563bf2a5aacf5ddafbdd7d67d0ffcdfd3e06fd443bfa6268389307c55fea57
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - HDF5
  - HDS
  - HDS (HDSv5 on HDF5)
  - NDF
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# memory-mapping-configuration-testing

## Summary

Test and benchmark the HDSv5 datMap/datUnmap memory-mapping emulation, where HDF5-backed HDS must mmap an anonymous memory area and copy data back on datUnmap instead of using native HDF5 mapping, and quantify its cost by re-running benchmark rows with mapping disabled.

## When to use

Use when porting or profiling HDS datMap/datUnmap semantics onto HDF5, or when benchmark timing ratios (v5/v4) are unexpectedly high for map-heavy routines and you need to isolate the mmap/copy-back overhead.

## When NOT to use

- When the file being opened is a native HDS v4 file — the hds-v5 library does not handle HDS v4 files and migration is explicitly out of scope
- When the analysis target is HDF5-native mapping features (compression-filter-aware mapping) — HDSv5's datMap cannot use them and only emulates mapping via anonymous mmap

## Inputs

- Starlink software stack built against hds-v5 (github.com/Starlink/hds-v5)
- starbench/specdre and starbench/kappa benchmark routines
- ccdpack and ccdbig benchmark configurations
- SCUBA-2 acquisition file exercising bulk data plus table structures written in a single operation

## Outputs

- Per-benchmark wall-clock timings in seconds for HDSv5 with and without memory mapping
- v5/v4 timing ratios per benchmark row (Table 3 comparison, table3_timings.csv)
- Verified datMap/datUnmap copy-back behavior for mapped locators

## How to apply

Rebuild the Starlink stack linked against Starlink/hds-v5 and confirm that datMap allocates an anonymous mmap area and that datUnmap copies data back to the underlying HDF5 dataset, since HDF5 does not support memory mapping (e.g. because of compression filters and type conversion). Ensure datGet/datPut detect a memory-mapped locator, datAnnul triggers the copy, and hdsClose/datAnnul at the file root closes all mmapped arrays. Then run the benchmark suites (starbench/specdre, starbench/kappa, ccdpack, ccdbig, and the SCUBA-2 orac-dr test) twice per row: once normally and once with memory mapping disabled (the asterisked rows), recording wall-clock seconds. Compute the v5/v4 timing ratio to attribute slowdowns to the mapping layer rather than HDF5 itself, expecting near-parity (within a few per cent) for reasonably-sized datasets.

## Related tools

- **HDS (HDSv5 on HDF5)** (Library under test: datMap/datUnmap emulate memory mapping over HDF5 datasets; exercised via the Starlink benchmarks) — https://github.com/Starlink
- **HDF5** (Underlying storage layer whose lack of native memory mapping forces the anonymous-mmap and copy-back implementation)
- **NDF** (Starlink N-dimensional data format whose applications run on top of HDS and exercise the mapping path in benchmarks)

## Evaluation signals

- Asterisked benchmark rows (mapping disabled) show lower wall-clock times than the default mapped runs, isolating mmap/copy-back overhead
- v5/v4 timing ratios for reasonably-sized dataset benchmarks are within a few per cent of 1.0, matching the reported finding
- datGet/datPut on a memory-mapped locator correctly detect or complain about the mapped state as specified in the README porting notes
- hdsClose (or datAnnul at the file root) closes all mmapped arrays; program termination before datUnmap does not silently lose data
- Double-mapped primitives (two locators mapping the same primitive) behave consistently with HDS semantics or are explicitly flagged

## Limitations

- datMap must copy data back on datUnmap; if a program ends before datUnmap (or before the datAnnul-equivalent at file root), unmapped changes may be lost
- Slices and vectorized locators complicate mapping semantics: datVec locators must report rank-1 vectorized dimensions, and datSlice/datCell dataspace selection interacts with mapped state
- The effort to support 64-bit dimension sizes and other HDSv4 parity issues is uncertain; a few of the ~150 HDS API functions querying low-level implementation details remain unimplemented

## Evidence

- [discussion] Benchmark parity expectation: [evidence span withheld — anchor status: not_found]
