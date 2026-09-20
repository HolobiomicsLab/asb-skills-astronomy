---
name: cuda-kernel-profiling-data-extraction
description: Use when reproducing or verifying GPU kernel profiling tables (occupancy,
  register usage, block size, instruction vs memory time, bandwidths) reported for
  specific CUDA kernels on an NVIDIA K40, per floating-point precision.
when_to_use_negative:
- The input is raw profiler binary output or a live profiling session rather than
  published table values to transcribe and validate
- The target kernels are not the Montblanc EK (per-antenna) or B Sum (per-baseline)
  kernels, so the expected register/thread/occupancy anchors do not apply
- Only a single precision is of interest and the float/double distinction in the source
  tables cannot be preserved
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Montblanc
  role: Source of the CUDA kernels (EK and B Sum) whose profiler results are extracted
    and tabulated
  repo: https://github.com/ska-sa/montblanc
- name: CUDA
  role: Architecture on which the kernels run and whose NVIDIA Profiler supplies registers/threads/occupancy
    and bandwidth metrics
- name: Python
  role: Implementation language used to serialize transcribed tabular records to CSV
    and JSON
  repo: https://github.com/numpy/numpy
- name: NVIDIA K40
  role: GPU hardware on which the profiled kernel configurations and bandwidths were
    measured
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.003
    title: 'Montblanc : GPU accelerated radio interferometer measurement equations
      in support of Bayesian inference for radio observations'
  source_content_hash: sha256:e977fc094cbb5335712f713b1687ac4137e1986d5dd874b44a7ada217413a8fd
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
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# cuda-kernel-profiling-data-extraction

## Summary

Extracts and tabulates NVIDIA Profiler metrics — registers per thread, threads per block, occupancy, compute/memory instruction split, and memory-hierarchy bandwidth (shared, global, L1, L2, DRAM) — for CUDA kernels (Montblanc's EK and B Sum) on an NVIDIA K40, separated by float and double precision.

## When to use

Use when reproducing or verifying GPU kernel profiling tables (occupancy, register usage, block size, instruction vs memory time, bandwidths) reported for specific CUDA kernels on an NVIDIA K40, per floating-point precision.

## When NOT to use

- The input is raw profiler binary output or a live profiling session rather than published table values to transcribe and validate
- The target kernels are not the Montblanc EK (per-antenna) or B Sum (per-baseline) kernels, so the expected register/thread/occupancy anchors do not apply
- Only a single precision is of interest and the float/double distinction in the source tables cannot be preserved

## Inputs

- Paper profiling-section tables of NVIDIA Profiler results for EK and B Sum kernels (configuration, timing, and bandwidth tables)
- NVIDIA Profiler output from running Montblanc's EK and B Sum CUDA kernels on an NVIDIA K40
- Kernel/precision keys (EK, B Sum; float, double)

## Outputs

- CSV keyed by kernel name and precision with registers per thread, threads per block, and occupancy
- CSV of instruction-time vs memory-time percentages per kernel per precision
- CSV of shared/global/L1/L2/DRAM bandwidth values per kernel per precision
- JSON with one record per kernel per precision containing all transcribed metrics

## How to apply

Locate the profiler results tables in the paper's profiling section for the EK kernel (per-antenna terms) and the B Sum kernel (per-baseline terms), covering both single and double precision. Transcribe kernel configuration values — registers per thread, threads per block, and achieved occupancy — into a CSV keyed by kernel name and precision, e.g. EK float: 31 registers/512 threads/87% occupancy, EK double: 44/256/59%; B Sum float: 46/256/60%, B Sum double: 63/128/50%. Next, transcribe the compute/memory split (percentage of instruction time versus memory time) and the memory-hierarchy bandwidths (shared, global, L1, L2, DRAM) in the profiler's reported units, preserving the float/double distinction. Finally, serialize the same records to JSON with one record per kernel per precision, so the tables can be validated programmatically. The rationale is that the dominant kernel appears theoretically memory bound, but profiling shows it balanced between compute and memory due to L1/L2 cache retention, so both timing and bandwidth columns must be captured to support that claim.

## Related tools

- **Montblanc** (Source of the CUDA kernels (EK and B Sum) whose profiler results are extracted and tabulated) — https://github.com/ska-sa/montblanc
- **CUDA** (Architecture on which the kernels run and whose NVIDIA Profiler supplies registers/threads/occupancy and bandwidth metrics)
- **Python** (Implementation language used to serialize transcribed tabular records to CSV and JSON) — https://github.com/numpy/numpy
- **NVIDIA K40** (GPU hardware on which the profiled kernel configurations and bandwidths were measured)

## Examples

```
import , csv; records=[{'kernel':'EK','precision':'float','registers':31,'threads_per_block':512,'occupancy':0.87},{'kernel':'EK','precision':'double','registers':44,'threads_per_block':256,'occupancy':0.59},{'kernel':'B Sum','precision':'float','registers':46,'threads_per_block':256,'occupancy':0.60},{'kernel':'B Sum','precision':'double','registers':63,'threads_per_block':128,'occupancy':0.50}]; .dump(records, open('profiler_tables.','w'))
```

## Evaluation signals

- Every transcribed value matches the paper's reported profiler number for the corresponding kernel, metric, and precision
- Both kernels (EK and B Sum) and both precisions (float and double) are present in the output tables
- Known anchor values reproduce exactly: EK float 31 registers/512 threads/87% occupancy; EK double 44/256/59%; B Sum float 46/256/60%; B Sum double 63/128/50%
- JSON serialization contains one record per kernel per precision, mirroring the CSV rows one-to-one
- Memory-hierarchy bandwidth rows include all five levels (shared, global, L1, L2, DRAM) in the profiler's reported units

## Limitations

- Values are specific to the NVIDIA K40 and Montblanc's kernel versions at the time of publication (2015); newer GPU architectures or Montblanc releases (which later moved to a Tensorflow implementation) will differ
- Only single-GPU profiling is covered; multi-GPU and clustered solutions were not yet available in Montblanc
- Profiling reflects the paper's configuration where analytic expressions were substituted for Eps (DDE) terms, so timings may not generalize to full DDE evaluation

## Evidence

- [results] Per the NVIDIA Profiler (Table 3): EK kernel uses 31 registers/512 threads/87% occupancy (float) and 44/256/59% (double); B Sum kernel uses 46/256/60% (float) and 63/128/50% (double).: [evidence span withheld — anchor status: not_found]
- [abstract] The dominant kernel is theoretically memory bound but balanced in practice.: "Theoretical analysis of Montblanc’s dominant CUDA kernel suggests that it is memory bound. In practice, profiling shows that is balanced between compute and memory, as much of the data required by"
- [discussion] EK kernel computes per-antenna terms and outputs them to GPU main memory.: [evidence span withheld — anchor status: not_found]
- [discussion] B Sum kernel reads per-antenna terms and combines them to form per-baseline terms.: [evidence span withheld — anchor status: not_found]
- [abstract] Montblanc is implemented as a Python package based on NVIDIA's CUDA architecture.: "We implemented Montblanc as a Python package based upon NVIDIA’s CUDA architecture."
