---
name: memory-hierarchy-bandwidth-tabulation
description: Use when reproducing or cross-checking GPU kernel profiling tables —
  registers, threads per block, occupancy, instruction-time splits, or per-level memory
  bandwidths — for CUDA kernels profiled on an NVIDIA K40 or similar device.
when_to_use_negative:
- The goal is theoretical bandwidth/occupancy analysis from kernel source rather than
  reproducing measured NVIDIA Profiler numbers
- The kernels profiled are not Montblanc's EK (per-antenna) and B Sum (per-baseline)
  kernels, so the kernel/precision record keys would not match
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: Montblanc
  role: Source of the EK and B Sum CUDA kernels whose profiler tables are tabulated
  repo: https://github.com/ska-sa/montblanc
- name: CUDA
  role: Architecture on which Montblanc is implemented and whose NVIDIA Profiler supplies
    the measured kernel metrics
- name: Python
  role: Language used to implement Montblanc and to serialize the tabulated records
    to CSV/JSON
- name: NVIDIA K40
  role: GPU hardware on which the EK and B Sum kernels are profiled
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.003
    title: 'Montblanc : GPU accelerated radio interferometer measurement equations
      in support of Bayesian inference for radio observations'
  source_content_hash: sha256:fe3fa5a1f86e3dfbab63b6675ecfab07015e9b313a99acf7be2f708c3fcf4e23
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ska-sa/montblanc
  tools:
  - CUDA
  - Montblanc
  - NVIDIA K40
  - Python
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# memory-hierarchy-bandwidth-tabulation

## Summary

Transcribes NVIDIA Profiler measurements of kernel configuration (registers/thread, threads/block, occupancy), compute-vs-memory instruction time, and memory-hierarchy bandwidths (shared, global, L1, L2, DRAM) for Montblanc's EK and B Sum CUDA kernels on the NVIDIA K40, per float/double precision, into CSV and JSON records.

## When to use

Use when reproducing or cross-checking GPU kernel profiling tables — registers, threads per block, occupancy, instruction-time splits, or per-level memory bandwidths — for CUDA kernels profiled on an NVIDIA K40 or similar device.

## When NOT to use

- The goal is theoretical bandwidth/occupancy analysis from kernel source rather than reproducing measured NVIDIA Profiler numbers
- The kernels profiled are not Montblanc's EK (per-antenna) and B Sum (per-baseline) kernels, so the kernel/precision record keys would not match

## Inputs

- NVIDIA Profiler output tables for the EK kernel and B Sum kernel (float and double precision) on the NVIDIA K40
- Paper profiling-section tables: kernel configuration (registers/thread, threads/block, occupancy), instruction time split, and memory-hierarchy bandwidths (shared, global, L1, L2, DRAM)

## Outputs

- CSV keyed by kernel name and precision with registers per thread, threads per block, and occupancy
- CSV of instruction-time percentages (compute vs memory) per kernel per precision
- CSV of memory-hierarchy bandwidths (shared, global, L1, L2, DRAM) per kernel per precision
- JSON with one record per kernel per precision containing the same tabulated fields

## How to apply

Locate the profiler results tables in the paper's profiling section for the EK kernel (per-antenna terms) and the B Sum kernel (per-baseline terms) for both float and double precision. Transcribe kernel configuration — registers per thread, threads per block, occupancy (EK: 31/512/87% float, 44/256/59% double; B Sum: 46/256/60% float, 63/128/50% double) — into a CSV keyed by kernel name and precision. Next transcribe the instruction-time percentage split between compute and memory, preserving the paper's finding that the theoretically memory-bound dominant kernel is balanced in practice due to L1/L2 cache retention. Then transcribe memory-hierarchy bandwidths (shared, global, L1, L2, DRAM) in the profiler's reported units, keeping the float/double distinction. Finally serialize the same records to JSON with one record per kernel per precision.

## Related tools

- **Montblanc** (Source of the EK and B Sum CUDA kernels whose profiler tables are tabulated) — https://github.com/ska-sa/montblanc
- **CUDA** (Architecture on which Montblanc is implemented and whose NVIDIA Profiler supplies the measured kernel metrics)
- **Python** (Language used to implement Montblanc and to serialize the tabulated records to CSV/JSON)
- **NVIDIA K40** (GPU hardware on which the EK and B Sum kernels are profiled)

## Evaluation signals

- Every transcribed value matches the paper's reported profiler number for the corresponding kernel, metric, and precision
- Both kernels (EK and B Sum) and both precisions (float, double) are present in the tables
- CSV and JSON serializations contain identical records — one record per kernel per precision
- EK float shows 31 registers/512 threads/87% occupancy and B Sum double shows 63 registers/128 threads/50% occupancy as spot-check anchors

## Limitations

- Values are specific to the NVIDIA K40 and Montblanc's kernel versions; they do not transfer to other GPUs or later Montblanc revisions
- The dominant CUDA kernel is theoretically memory bound but measured as balanced between compute and memory because of L1/L2 cache retention, so naive theoretical predictions will disagree with the tabulated measurements

## Evidence

- [results] Per the NVIDIA Profiler (Table 3): EK kernel uses 31 registers/512 threads/87% occupancy (float) and 44/256/59% (double); B Sum kernel uses 46/256/60% (float) and 63/128/50% (double).: [evidence span withheld — anchor status: not_found]
- [discussion] Expensive, per-antenna terms are computed in an EK kernel, and output to the GPU's main memory. These per-antenna terms are read in by a B Sum kernel: [evidence span withheld — anchor status: not_found]
