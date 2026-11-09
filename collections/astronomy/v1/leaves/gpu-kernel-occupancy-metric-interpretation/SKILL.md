---
name: gpu-kernel-occupancy-metric-interpretation
description: Use when you have NVIDIA Profiler output for CUDA kernels and need to
  transcribe and interpret per-kernel register usage, block size, and achieved occupancy,
  or to reconcile theoretical memory-boundness with profiled compute/memory behavior.
when_to_use_negative:
- The kernels run on hardware whose register/SM limits differ from the K40, so the
  transcribed occupancy figures would not transfer
- The goal is simulator-level performance comparison (e.g. Montblanc vs MeqTrees or
  OSKAR timings) rather than kernel-level occupancy interpretation
- Only one kernel or one precision is available, so the float/double and EK/B Sum
  comparisons the skill depends on cannot be made
tools:
- name: Montblanc
  role: The GPU RIME implementation whose EK and B Sum CUDA kernels are profiled for
    occupancy metrics
  repo: https://github.com/ska-sa/montblanc
- name: CUDA
  role: Architecture on which Montblanc is implemented; supplies the profiler and
    kernel-level metrics (registers, threads per block, occupancy)
- name: NVIDIA K40
  role: Target GPU on which the EK and B Sum kernels are profiled in both float and
    double precision
- name: Python
  role: Language in which Montblanc is packaged; used to serialize the transcribed
    profiler tables to CSV and JSON
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.003
    title: 'Montblanc : GPU accelerated radio interferometer measurement equations
      in support of Bayesian inference for radio observations'
  source_content_hash: sha256:98138f4743a1fb6ad0885c1ca4434a4b6630d9a74f78985cdc03e2cad9acf8af
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ska-sa/montblanc
  tools:
  - CUDA
  - Montblanc
  - NVIDIA K40
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# gpu-kernel-occupancy-metric-interpretation

## Summary

Method for reading and interpreting NVIDIA Profiler occupancy metrics (registers per thread, threads per block, occupancy percentage) for CUDA kernels, as applied to Montblanc's EK and B Sum RIME kernels on an NVIDIA K40 in both float and double precision.

## When to use

Use when you have NVIDIA Profiler output for CUDA kernels and need to transcribe and interpret per-kernel register usage, block size, and achieved occupancy, or to reconcile theoretical memory-boundness with profiled compute/memory behavior.

## When NOT to use

- The kernels run on hardware whose register/SM limits differ from the K40, so the transcribed occupancy figures would not transfer
- The goal is simulator-level performance comparison (e.g. Montblanc vs MeqTrees or OSKAR timings) rather than kernel-level occupancy interpretation
- Only one kernel or one precision is available, so the float/double and EK/B Sum comparisons the skill depends on cannot be made

## Inputs

- NVIDIA Profiler output tables for CUDA kernels (occupancy, instruction/memory time split, memory-hierarchy bandwidths)
- Kernel configuration parameters (registers per thread, threads per block) per precision (float and double)
- Target GPU hardware specification (NVIDIA K40)

## Outputs

- CSV/JSON records keyed by kernel name (EK, B Sum) and precision with registers/thread, threads/block, and occupancy
- Per-kernel compute (instruction) vs memory time percentages
- Per-kernel memory-hierarchy bandwidth values (shared, global, L1, L2, DRAM) with units preserved

## How to apply

Run the NVIDIA Profiler on the target GPU (here the NVIDIA K40) over each CUDA kernel of interest — in Montblanc's case the EK kernel (per-antenna terms) and the B Sum kernel (per-baseline terms) — separately for float and double precision. Transcribe, per kernel and per precision, the registers per thread, threads per block, and achieved occupancy: EK uses 31 registers/512 threads/87% occupancy in float and 44 registers/256 threads/59% in double; B Sum uses 46/256/60% in float and 63/128/50% in double. Next, transcribe the instruction-time versus memory-time percentage split and the memory-hierarchy bandwidths (shared, global, L1, L2, DRAM) in the profiler's reported units, preserving the float/double distinction. Interpret the occupancy numbers against the register/threads-per-block configuration to judge whether the kernel is occupancy-limited, and compare the theoretical analysis (Montblanc's dominant kernel is theoretically memory bound) against the profiled instruction/memory balance, which showed the kernel to be balanced due to L1/L2 cache retention. Validate that every transcribed value matches the profiler table for the corresponding kernel, metric, and precision, with both kernels and both precisions present.

## Related tools

- **Montblanc** (The GPU RIME implementation whose EK and B Sum CUDA kernels are profiled for occupancy metrics) — https://github.com/ska-sa/montblanc
- **CUDA** (Architecture on which Montblanc is implemented; supplies the profiler and kernel-level metrics (registers, threads per block, occupancy))
- **NVIDIA K40** (Target GPU on which the EK and B Sum kernels are profiled in both float and double precision)
- **Python** (Language in which Montblanc is packaged; used to serialize the transcribed profiler tables to CSV and JSON)

## Evaluation signals

- Every transcribed value matches the paper's reported profiler number for the corresponding kernel, metric, and precision
- Both kernels (EK and B Sum) and both precisions (float and double) are present in the occupancy tables
- Recorded configuration matches expected values: EK 31 registers/512 threads/87% (float) and 44/256/59% (double); B Sum 46/256/60% (float) and 63/128/50% (double)
- Compute/memory time split supports the reported conclusion that the theoretically memory-bound kernel is balanced in practice
- JSON serialization reproduces the same tabulated records with one record per kernel per precision

## Limitations

- The dominant CUDA kernel is theoretically memory bound, but profiling shows it is balanced between compute and memory because much of the data is retained in L1/L2 cache — theoretical occupancy/bandwidth analysis alone can mispredict behavior
- Metrics are specific to the NVIDIA K40; register counts, threads per block, and occupancy percentages do not generalize to other GPU architectures
- Only single GPUs are currently supported in Montblanc, so multi-GPU or clustered occupancy behavior is not covered

## Evidence

- [abstract] Theoretical analysis of Montblanc's dominant CUDA kernel suggests that it is memory bound. In practice, profiling shows that is balanced between compute and memory, as much of the data required by: [evidence span withheld — anchor status: not_found]
- [discussion] Expensive, per-antenna terms are computed in an EK kernel, and output to the GPU's main memory.: [evidence span withheld — anchor status: not_found]
- [discussion] These per-antenna terms are read in by a B Sum kernel, and combined to form per-baseline terms.: [evidence span withheld — anchor status: not_found]
