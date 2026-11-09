---
name: performance-comparison-reporting
description: Use when measuring and contrasting elapsed load time and resident memory
  between two I/O configurations (e.g. anonymous mmap via datMap vs direct HDF5 dataset
  access) for the same large dataset in one application.
when_to_use_negative:
- Datasets are small enough that load time and memory differences between configurations
  are negligible ('reasonably-sized datasets')
- The comparison target is file size or write throughput rather than application-level
  load time and resident memory
- The application supports native HDF5 memory mapping, so the anonymous-mmap emulation
  path is not exercised
tools:
- name: GAIA
  role: Visualization tool in which the 4 GB dataset is loaded to measure display
    load time and peak process memory under each configuration
- name: HDS
  role: Starlink Hierarchical Data System API layer whose HDSv5 implementation on
    HDF5 is being benchmarked
  repo: https://github.com/Starlink
- name: HDF5
  role: Underlying storage layer providing direct dataset access in the unmapped configuration
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.02.003
    title: Reimplementing the Hierarchical Data System using HDF5
  source_content_hash: sha256:ac049b54caf0d5a8f9a4c174e1dd22b2d456a7e58e710b387253755c9b7575be
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GAIA
  - HDF5
  - HDS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# performance-comparison-reporting

## Summary

Benchmark and side-by-side report the load time and peak resident memory of two access configurations of a data library, as done when comparing HDSv5 memory mapping versus direct HDF5 access while loading a 4 GB dataset into the GAIA visualization tool.

## When to use

Use when measuring and contrasting elapsed load time and resident memory between two I/O configurations (e.g. anonymous mmap via datMap vs direct HDF5 dataset access) for the same large dataset in one application.

## When NOT to use

- Datasets are small enough that load time and memory differences between configurations are negligible ('reasonably-sized datasets')
- The comparison target is file size or write throughput rather than application-level load time and resident memory
- The application supports native HDF5 memory mapping, so the anonymous-mmap emulation path is not exercised

## Inputs

- 4 GB dataset in HDSv5/HDF5 format openable via the HDS API layer
- GAIA visualization tool (Draper et al., 2009, ascl:1403.024)
- Tuning parameter controlling memory mapping (mapped vs unmapped access)

## Outputs

- gaia_load_benchmark.csv with elapsed load seconds and peak resident memory per configuration
- Side-by-side comparison of mapped vs unmapped load time and memory usage

## How to apply

Obtain the dataset in the library's native format (here HDSv5/HDF5, 4 GB) and open it through the target application (GAIA, Draper et al., 2009, ascl:1403.024) via the HDF5-backed HDS API layer. Run the first configuration with memory mapping enabled, where datMap mmaps an anonymous memory area and datUnmap copies data back to the underlying dataset, recording display load time and peak process memory. Repeat with memory mapping disabled (direct HDF5 dataset access) and record the same metrics. Tabulate elapsed seconds and resident memory side by side (e.g. gaia_load_benchmark.csv); the mapped run should load in a few seconds using tens of MB, the unmapped run roughly an order of magnitude slower and near the full dataset size (~5 GB). Rationale: HDF5 does not natively support memory mapping (compression filters, type conversion), so mapping in HDSv5 is an anonymous-mmap emulation whose cost profile differs sharply from direct reads, and this contrast is what the report must capture.

## Related tools

- **GAIA** (Visualization tool in which the 4 GB dataset is loaded to measure display load time and peak process memory under each configuration)
- **HDS** (Starlink Hierarchical Data System API layer whose HDSv5 implementation on HDF5 is being benchmarked) — https://github.com/Starlink
- **HDF5** (Underlying storage layer providing direct dataset access in the unmapped configuration)

## Evaluation signals

- Mapped run load time is on the order of a few seconds with peak memory in the tens of MB
- Unmapped run takes roughly an order of magnitude longer (~10 s) and consumes memory comparable to the full dataset size (~5 GB)
- Both configurations load the identical 4 GB HDSv5/HDF5 dataset, so only the access mode differs
- CSV contains two rows/configurations with both elapsed seconds and resident-memory columns populated

## Limitations

- HDF5 does not support memory mapping natively (compression filters, type conversion), so the mapped path is an anonymous-mmap emulation; datGet/datPut must notice a mapped locator and datAnnul/hdsClose must handle copy-back
- Behavior is undefined if the program ends before datUnmap runs, or if the primitive is mapped twice with different locators
- Results reflect the Section 5.9 contrast for a 4 GB dataset and may not transfer to reasonably-sized datasets or other applications

## Evidence

- [intro] a 4 GB dataset was loaded into the GAIA visualization tool (Draper et al., 2009, ascl:1403.024): [evidence span withheld — anchor status: not_found]
