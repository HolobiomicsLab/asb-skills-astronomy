---
name: process-memory-profiling
description: Use when comparing load-time and peak-memory costs of opening GB-scale
  HDS/HDF5 datasets in a tool, specifically to quantify the effect of enabling vs.
  disabling memory mapping (datMap/datUnmap) on the HDS API layer.
when_to_use_negative:
- The dataset is small enough that mapped vs. unmapped differences are not measurable
  (the paper notes HDSv5 performs as well as HDSv4 in most tests involving reasonably-sized
  datasets)
- The goal is to profile compression or type-conversion overhead rather than the mmap/direct-read
  load-time and memory contrast
- The target tool does not access data through the HDS API layer, so datMap/datUnmap
  semantics do not apply
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: GAIA
  role: Visualization tool used as the test consumer whose display load time and peak
    process memory are measured when loading the 4 GB dataset via HDSv5
- name: HDS
  role: Starlink Hierarchical Data System API layer (HDSv5) whose datMap/datUnmap
    memory-mapping behavior is toggled during the benchmark
  repo: https://github.com/Starlink
- name: HDF5
  role: Underlying storage library providing direct dataset access in the memory-mapping-disabled
    configuration
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.02.003
    title: Reimplementing the Hierarchical Data System using HDF5
  source_content_hash: sha256:cd9cb382c3a5931ad1db898bc347173f6e93d29dd0074dd5127a2b4b17bab3b3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GAIA
  - HDF5
  - HDS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# process-memory-profiling

## Summary

Benchmarks the elapsed load time and peak resident memory of a process (e.g., the GAIA visualization tool) when opening a large dataset through HDSv5/HDF5, contrasting memory-mapped access against direct dataset reads. Rationale: the paper uses this contrast (Section 5.9) to show that datMap's anonymous-mmap strategy keeps memory use in the tens of MB while direct access consumes memory comparable to the full dataset size.

## When to use

Use when comparing load-time and peak-memory costs of opening GB-scale HDS/HDF5 datasets in a tool, specifically to quantify the effect of enabling vs. disabling memory mapping (datMap/datUnmap) on the HDS API layer.

## When NOT to use

- The dataset is small enough that mapped vs. unmapped differences are not measurable (the paper notes HDSv5 performs as well as HDSv4 in most tests involving reasonably-sized datasets)
- The goal is to profile compression or type-conversion overhead rather than the mmap/direct-read load-time and memory contrast
- The target tool does not access data through the HDS API layer, so datMap/datUnmap semantics do not apply

## Inputs

- 4 GB dataset in HDSv5/HDF5 format (openable via the HDF5-backed HDS API layer)
- GAIA visualization tool (Draper et al., 2009, ascl:1403.024) build with HDSv5 library
- Tuning parameter toggling memory mapping on/off in the HDSv5 library

## Outputs

- gaia_load_benchmark.csv tabulating load time (seconds) and peak resident memory for mapped vs. unmapped configurations
- Recorded load time and peak process memory per configuration (mapped: few seconds / tens of MB; unmapped: ~10 s / ~5 GB)

## How to apply

Obtain a large dataset in HDSv5/HDF5 format (the paper uses a 4 GB dataset loaded into GAIA, Draper et al. 2009, ascl:1403.024, via the HDF5-backed HDS API). Run the tool with memory mapping enabled, where datMap mmaps an anonymous memory area and datUnmap copies the data back to the underlying HDF5 dataset, recording display load time and peak process memory. Repeat the load with memory mapping disabled (direct HDF5 dataset access) and record the same metrics. Tabulate the two configurations side by side (elapsed seconds and resident memory, e.g. into gaia_load_benchmark.csv). Decision point: if the unmapped run takes roughly an order of magnitude longer and consumes memory comparable to the full dataset size, the mmap advantage is reproduced.

## Related tools

- **GAIA** (Visualization tool used as the test consumer whose display load time and peak process memory are measured when loading the 4 GB dataset via HDSv5)
- **HDS** (Starlink Hierarchical Data System API layer (HDSv5) whose datMap/datUnmap memory-mapping behavior is toggled during the benchmark) — https://github.com/Starlink
- **HDF5** (Underlying storage library providing direct dataset access in the memory-mapping-disabled configuration)

## Evaluation signals

- Mapped run: load time on the order of a few seconds with peak memory in the tens of MB
- Unmapped run: load time roughly an order of magnitude longer (~10 s) with memory use comparable to the full dataset size (~5 GB for the 4 GB dataset)
- The benchmark CSV contains two rows (mapped and unmapped) each with elapsed-seconds and resident-memory columns populated
- Measured metrics are consistent across repeated runs of the same configuration

## Limitations

- HDF5 does not natively support memory mapping, so HDSv5 emulates it via anonymous mmap plus copy-back on datUnmap, adding copy overhead on unmap
- Behavior when a program ends before datUnmap, when a primitive is mapped twice with different locators, and cleanup of mmapped arrays at hdsClose/datAnnul were still open questions in the implementation
- HDSv5 performance parity was only demonstrated for reasonably-sized datasets; GB-scale behavior is dominated by the mmap vs. direct-read distinction

## Evidence

- [intro] a 4 GB dataset was loaded into the GAIA visualization tool (Draper et al., 2009, ascl:1403.024): [evidence span withheld — anchor status: not_found]
- [discussion] HDSv5 performs comparably to HDSv4 on reasonably-sized datasets: [evidence span withheld — anchor status: not_found]
