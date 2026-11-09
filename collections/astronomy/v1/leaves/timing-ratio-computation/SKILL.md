---
name: timing-ratio-computation
description: Use when two builds of the same data-access library (e.g. native HDSv4
  and the HDF5-backed hds-v5) have been benchmarked on identical routines and you
  must assemble a seconds-based timing table and v5/v4 ratio comparison to assess
  whether the replacement performs comparably.
when_to_use_negative:
- The goal is a functional (correctness) comparison of file formats rather than a
  performance comparison — the paper separately compares file sizes.
- Only one library build has been benchmarked, so no v5/v4 ratio can be computed.
- The task involves migrating HDS v4 files to HDF5, which the hds-v5 implementation
  explicitly does not yet address.
tools:
- name: HDS
  role: Native Hierarchical Data System (HDSv4) against which the HDF5-backed build
    is benchmarked
  repo: https://github.com/Starlink
- name: HDF5
  role: Storage layer underlying the hds-v5 replacement library whose timings are
    compared to HDSv4
  repo: https://github.com/Starlink/hds-v5
- name: NDF
  role: Starlink N-Dimensional Data Format whose access path through the HDS library
    is exercised by the benchmark routines
  repo: https://github.com/Starlink
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.02.003
    title: Reimplementing the Hierarchical Data System using HDF5
  source_content_hash: sha256:aa3407d4e62a5ab6f81ac4c8684e3da471ede5db5e33cd1842549f0e80d53b19
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - HDF5
  - HDS
  - NDF
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# timing-ratio-computation

## Summary

Computes per-benchmark v5/v4 wall-clock timing ratios to compare an HDF5-backed library implementation against its native predecessor, as done when benchmarking HDSv5 against HDSv4 using the Starlink benchmark suites (starbench, ccdpack, ccdbig) and a SCUBA-2 orac-dr data reduction test.

## When to use

Use when two builds of the same data-access library (e.g. native HDSv4 and the HDF5-backed hds-v5) have been benchmarked on identical routines and you must assemble a seconds-based timing table and v5/v4 ratio comparison to assess whether the replacement performs comparably.

## When NOT to use

- The goal is a functional (correctness) comparison of file formats rather than a performance comparison — the paper separately compares file sizes.
- Only one library build has been benchmarked, so no v5/v4 ratio can be computed.
- The task involves migrating HDS v4 files to HDF5, which the hds-v5 implementation explicitly does not yet address.

## Inputs

- Wall-clock timing measurements (seconds) from the starbench/specdre and starbench/kappa benchmark routines under HDSv4 and hds-v5 builds
- Wall-clock timings (seconds) from the ccdpack and ccdbig benchmarks under both builds
- Timing from the SCUBA-2 orac-dr data reduction test on the SCUBA-2 acquisition file under both builds
- Asterisked benchmark rows re-run with memory mapping disabled

## Outputs

- table3_timings.csv containing per-benchmark timings in seconds for HDSv4 and HDSv5 plus v5/v4 ratios
- Per-row v5/v4 timing ratios for starbench, ccdpack, ccdbig, and the orac-dr test

## How to apply

Build the Starlink software stack twice, once linked against native HDS (HDSv4) and once against hds-v5 (HDF5-backed), then run the starbench/specdre and starbench/kappa routines, the ccdpack and ccdbig benchmarks, and the SCUBA-2 orac-dr reduction on a SCUBA-2 acquisition file under both builds, recording wall-clock times in seconds. Repeat the asterisked benchmark rows with memory mapping disabled, because HDSv5's datMap must mmap an anonymous memory area and datUnmap must copy data back rather than using native HDF5 mapping. For each benchmark row divide the v5/v4 times to obtain the timing ratio and assemble the Table 3 comparison in seconds. Judge correctness by checking that ratios are near 1 for real-world tasks with larger datasets, consistent with the finding that the HDF5-backed library performs as well as HDSv4 in most tests involving reasonably-sized datasets.

## Related tools

- **HDS** (Native Hierarchical Data System (HDSv4) against which the HDF5-backed build is benchmarked) — https://github.com/Starlink
- **HDF5** (Storage layer underlying the hds-v5 replacement library whose timings are compared to HDSv4) — https://github.com/Starlink/hds-v5
- **NDF** (Starlink N-Dimensional Data Format whose access path through the HDS library is exercised by the benchmark routines) — https://github.com/Starlink

## Examples

```
python -c "import pandas as pd; t=pd.read_csv('table3_timings.csv'); t['v5_over_v4']=t['v5_s']/t['v4_s']; print(t)"
```

## Evaluation signals

- v5/v4 ratios for real-world tasks with larger datasets (e.g. the SCUBA-2 orac-dr test) should be close to 1, within a few per cent.
- Tasks using lots of small files with lots of I/O (e.g. ccdpack/ccdbig-style rows) should show HDSv4 much faster than HDSv5.
- Asterisked rows must be present in the table with both memory-mapped and memory-mapping-disabled runs, since datMap forces anonymous mmap with copy-back on datUnmap.
- All timings recorded in seconds and each ratio computed from the same benchmark run pair; the table assembled as the Table 3 comparison.
- Overall result should be consistent with the reported finding that HDSv5 performs as well as HDSv4 in most tests involving reasonably-sized datasets.

## Limitations

- HDSv5's datMap cannot use native HDF5 mapping; it must mmap an anonymous memory area and datUnmap must copy the data back, inflating timings for mapped operations unless those rows are re-run with memory mapping disabled.
- Small-file, small-I/O workloads are inherently disadvantageous to the HDF5-backed implementation, so ratios from such rows must not be read as general performance.
- HDF5 does not support memory mapping (presumably because of compression filters and data type conversion), so any mapped-array benchmark is testing the HDSv5 emulation path, not native HDF5 behaviour.

## Evidence

- [discussion] performs as well as HDSv4 on reasonably-sized datasets: [evidence span withheld — anchor status: not_found]
