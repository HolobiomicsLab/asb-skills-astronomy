---
name: hds-hdf5-library-benchmark-execution
description: Use when comparing wall-clock performance of two drop-in file-format
  library builds — native HDSv4 versus the HDF5-backed hds-v5 — using the Starlink
  starbench, ccdpack, ccdbig and SCUBA-2 orac-dr benchmarks.
when_to_use_negative:
- The goal is comparing output file sizes between HDSv4 and HDSv5 — that is the paper's
  separate size comparison, not benchmark timing execution.
- The library under test is not a drop-in replacement for the HDS API (locators, datMap/datUnmap,
  HDS groups); these benchmarks assume compatibility with the implemented HDS API
  functions.
- You simply need to read/write HDF5 files with general-purpose tools such as h5py
  and have no legacy HDS/NDF data or Starlink stack to evaluate.
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: HDS (HDSv4)
  role: Baseline native Hierarchical Data System that one Starlink build links against,
    supplying the reference timings for every benchmark row
  repo: https://github.com/Starlink
- name: hds-v5
  role: HDF5-backed drop-in HDS API replacement that the second Starlink build links
    against; its wall-clock times yield the v5/v4 ratios
  repo: https://github.com/Starlink/hds-v5
- name: HDF5
  role: Storage layer beneath hds-v5; its lack of memory-mapping support forces the
    datMap/datUnmap anonymous-mmap and copy-back path that is re-tested with mapping
    disabled
- name: NDF
  role: Extensible N-dimensional data format built on HDS whose processing tasks (e.g.,
    the orac-dr reduction) exercise the library during benchmarking
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.02.003
    title: Reimplementing the Hierarchical Data System using HDF5
  source_content_hash: sha256:62289aa4e9400c8b2e8c91b653e204c88aa343522639045559fdb7cdbbcfa5e3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - HDF5
  - HDS
  - HDS (HDSv4)
  - NDF
  - hds-v5
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# hds-hdf5-library-benchmark-execution

## Summary

Executes the Starlink benchmark suite (starbench/specdre, starbench/kappa, ccdpack, ccdbig and the SCUBA-2 orac-dr reduction) against two builds of the Starlink stack — one linked to native HDSv4 and one to the HDF5-backed hds-v5 library — to produce the Table 3 wall-clock comparison with per-row v5/v4 ratios.

## When to use

Use when comparing wall-clock performance of two drop-in file-format library builds — native HDSv4 versus the HDF5-backed hds-v5 — using the Starlink starbench, ccdpack, ccdbig and SCUBA-2 orac-dr benchmarks.

## When NOT to use

- The goal is comparing output file sizes between HDSv4 and HDSv5 — that is the paper's separate size comparison, not benchmark timing execution.
- The library under test is not a drop-in replacement for the HDS API (locators, datMap/datUnmap, HDS groups); these benchmarks assume compatibility with the implemented HDS API functions.
- You simply need to read/write HDF5 files with general-purpose tools such as h5py and have no legacy HDS/NDF data or Starlink stack to evaluate.

## Inputs

- Starlink software stack built and linked against the native HDS (HDSv4) library
- Starlink software stack built and linked against the hds-v5 (HDF5-backed) library from github.com/Starlink/hds-v5
- starbench/specdre and starbench/kappa benchmark routines
- ccdpack and ccdbig benchmark tasks
- SCUBA-2 acquisition file for the orac-dr data reduction test

## Outputs

- Wall-clock benchmark times in seconds per routine for both library builds
- Per-row v5/v4 timing ratios forming the Table 3 comparison
- table3_timings.csv with per-benchmark seconds and v5/v4 ratios

## How to apply

Build the Starlink software collection from github.com/Starlink twice: once linked against the native HDS (HDSv4) and once against the hds-v5 (HDF5-backed) library, so that hds-v5 acts as a drop-in HDS API replacement. Execute starbench/specdre and starbench/kappa, then ccdpack and ccdbig, under both builds and record wall-clock times in seconds. Run the SCUBA-2 orac-dr data reduction test on the SCUBA-2 acquisition file, which exercises bulk data plus table structures written in a single operation. Repeat the asterisked benchmark rows with memory mapping disabled, because HDSv5's datMap must mmap an anonymous memory area and datUnmap must copy data back to the underlying dataset (HDF5 does not support mapping), which otherwise penalises mapped-access rows. Compute the v5/v4 timing ratio per row, assemble the Table 3 comparison in seconds and write it to table3_timings.csv, interpreting ratios against the finding that HDSv5 performs within a few per cent of HDSv4 for reasonably-sized real-world datasets but is much slower for many-small-file, I/O-heavy tasks.

## Related tools

- **HDS (HDSv4)** (Baseline native Hierarchical Data System that one Starlink build links against, supplying the reference timings for every benchmark row) — https://github.com/Starlink
- **hds-v5** (HDF5-backed drop-in HDS API replacement that the second Starlink build links against; its wall-clock times yield the v5/v4 ratios) — https://github.com/Starlink/hds-v5
- **HDF5** (Storage layer beneath hds-v5; its lack of memory-mapping support forces the datMap/datUnmap anonymous-mmap and copy-back path that is re-tested with mapping disabled)
- **NDF** (Extensible N-dimensional data format built on HDS whose processing tasks (e.g., the orac-dr reduction) exercise the library during benchmarking)

## Examples

```
python -c "import pandas as pd; t=pd.read_csv('table3_timings.csv'); print(t.assign(ratio=t['v5']/t['v4']))"
```

## Evaluation signals

- Each Table 3 row has wall-clock seconds from both the HDSv4-linked and hds-v5-linked builds built from the same Starlink source tree.
- Asterisked (mapped-access) rows were re-run with memory mapping disabled, and their v5/v4 ratios drop once the datMap/datUnmap mmap-and-copy-back overhead is removed.
- Computed v5/v4 ratios match the reported pattern: within a few per cent for reasonably-sized datasets (e.g., orac-dr) and HDSv4 much faster for small-file, I/O-heavy benchmarks.
- table3_timings.csv exists, with per-benchmark seconds for both builds and a populated v5/v4 ratio column.
- Both builds run the benchmark routines to completion without HDS API errors, since all but a handful of the ~150 HDS API functions are implemented in hds-v5.

## Limitations

- HDF5 does not support memory mapping, so hds-v5's datMap must mmap an anonymous memory area and datUnmap copies data back to the dataset — mapped-access benchmarks overstate HDSv5 cost unless mapping is disabled.
- HDSv5 is much slower than HDSv4 for tasks using lots of small files with lots of I/O.
- hds-v5 chunks datasets based on the size of the array being created (H5S_UNLIMITED maxima are used to allow reshaping), with no HDS API knob to tune chunking, so timings depend on that default.
- Migration of existing HDS v4 files to HDF5 is not addressed; the benchmark compares library execution only, not file conversion cost.

## Evidence

- [intro] SCUBA-2 orac-dr test workload: [evidence span withheld — anchor status: not_found]
- [discussion] HDSv5 performance parity finding: [evidence span withheld — anchor status: not_found]
- [discussion] Source availability for building both stacks: [evidence span withheld — anchor status: not_found]
