---
name: rime-kernel-pipeline-context-understanding
description: Reference for interpreting Montblanc CUDA kernel profiling (EK and B
  Sum kernels) on an NVIDIA K40, or when structuring RIME evaluation into per-antenna
  and per-baseline GPU kernels before collecting occupancy or bandwidth metrics.
when_to_use_negative:
- The goal is to extend Montblanc to full DIE (G_ps) matrix evaluation or DDE E_ps
  terms — these are explicitly substituted with analytic expressions and not implemented,
  so kernel-level profiling context does not apply
- The target is multi-GPU or clustered execution — only single GPUs are currently
  supported and out-of-core BIRO is not yet implemented
- The task is CPU-side RIME evaluation (e.g. MeqTrees) rather than the Montblanc CUDA
  kernel pipeline on an NVIDIA GPU
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3398
tools:
- name: Montblanc
  role: GPU implementation of the RIME whose EK and B Sum CUDA kernels are the objects
    of the pipeline understanding and profiling
  repo: https://github.com/ska-sa/montblanc
- name: CUDA
  role: NVIDIA architecture on which Montblanc is implemented; the profiler metrics
    (occupancy, registers, bandwidth) are CUDA-specific
- name: Python
  role: Language of the Montblanc package; profiling records are serialized to CSV/JSON
    with Python
- name: NVIDIA K40
  role: Reference GPU on which the occupancy, compute/memory split and bandwidth figures
    are measured
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.003
    title: 'Montblanc : GPU accelerated radio interferometer measurement equations
      in support of Bayesian inference for radio observations'
  source_content_hash: sha256:dba6e71fc5704442f1a73e8339b51cbf5732530451c47f674ee9e49036e38225
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ska-sa/montblanc
  tools:
  - CUDA
  - Montblanc
  - NVIDIA K40
  - Python
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3398
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# rime-kernel-pipeline-context-understanding

## Summary

Understands the two-kernel GPU pipeline in Montblanc's RIME evaluation — the EK kernel computing per-antenna terms and the B Sum kernel combining them into per-baseline terms — as a prerequisite for interpreting or reproducing NVIDIA Profiler occupancy, compute/memory-split and memory-bandwidth results on the K40.

## When to use

Reference for interpreting Montblanc CUDA kernel profiling (EK and B Sum kernels) on an NVIDIA K40, or when structuring RIME evaluation into per-antenna and per-baseline GPU kernels before collecting occupancy or bandwidth metrics.

## When NOT to use

- The goal is to extend Montblanc to full DIE (G_ps) matrix evaluation or DDE E_ps terms — these are explicitly substituted with analytic expressions and not implemented, so kernel-level profiling context does not apply
- The target is multi-GPU or clustered execution — only single GPUs are currently supported and out-of-core BIRO is not yet implemented
- The task is CPU-side RIME evaluation (e.g. MeqTrees) rather than the Montblanc CUDA kernel pipeline on an NVIDIA GPU

## Inputs

- Montblanc Python/CUDA package (github.com/ska-sa/montblanc) running on an NVIDIA K40 GPU
- RIME sky-model parameters (lm coordinates, frequency, point/gaussian stokes, alpha, gaussian shape data sources)
- NVIDIA Profiler output for the EK and B Sum kernels, per precision (float/double)

## Outputs

- Kernel configuration table (registers per thread, threads per block, occupancy) keyed by kernel name and precision
- Compute/memory time-split table (percentage of instruction time vs memory time per kernel per precision)
- Memory-hierarchy bandwidth records (shared, global, L1, L2, DRAM) per kernel per precision, serializable as CSV or JSON with one record per kernel per precision

## How to apply

First map the pipeline: the EK kernel computes expensive per-antenna terms and writes them to GPU main memory; the B Sum kernel then reads those terms and combines them into per-baseline terms, from which model visibilities and a chi-squared value are produced for BIRO's Bayesian inference. Recognize that per-antenna (EK) and per-baseline (B Sum) workloads have different parallelism dimensions — over time, antennas, baselines, channels and sources — which is why the RIME is well suited to GPU implementation. When profiling, distinguish kernel by name and precision (float vs double), and record registers per thread, threads per block and occupancy for each (e.g. EK: 31 registers/512 threads/87% occupancy float, 44/256/59% double; B Sum: 46/256/60% float, 63/128/50% double). Also record the instruction-vs-memory time split and the memory-hierarchy bandwidths (shared, global, L1, L2, DRAM), noting that the theoretically memory-bound dominant kernel is balanced between compute and memory in practice due to L1/L2 cache retention. If the problem exceeds the GPU memory budget, remember that the RIME problem is subdivided along the time dimension using multiple solvers with asynchronous transfers, which affects what a profiler run measures.

## Related tools

- **Montblanc** (GPU implementation of the RIME whose EK and B Sum CUDA kernels are the objects of the pipeline understanding and profiling) — https://github.com/ska-sa/montblanc
- **CUDA** (NVIDIA architecture on which Montblanc is implemented; the profiler metrics (occupancy, registers, bandwidth) are CUDA-specific)
- **Python** (Language of the Montblanc package; profiling records are serialized to CSV/JSON with Python)
- **NVIDIA K40** (Reference GPU on which the occupancy, compute/memory split and bandwidth figures are measured)

## Examples

```
python scripts/profile_kernels.py --gpu K40 --kernels EK,BSum --precisions float,double --output profiler_tables.csv
```

## Evaluation signals

- Every transcribed profiler value matches the paper's reported number for the corresponding kernel, metric and precision (e.g. EK float = 31 registers, 512 threads, 87% occupancy; EK double = 44/256/59%; B Sum float = 46/256/60%; B Sum double = 63/128/50%)
- Both kernels (EK and B Sum) and both precisions (float and double) are present in the transcribed tables
- Units of memory-hierarchy bandwidth values (shared, global, L1, L2, DRAM) are preserved exactly as reported by the profiler
- The CSV and JSON serializations contain identical records, with one record per kernel per precision
- The compute/memory split is consistent with the paper's finding that the theoretically memory-bound dominant kernel is balanced between compute and memory in practice

## Limitations

- Analytic expressions were substituted for the E_ps (DDE) terms, so the profiled kernels do not evaluate full direction-dependent-effect matrices
- Evaluation of G_ps (DIE) matrices is excluded because BIRO operates on fully calibrated data
- Only single-GPU execution is supported; multi-GPU and clustered solutions, and an out-of-core BIRO solution, are not yet implemented
- Profiling results are specific to the NVIDIA K40 and the float/double distinction; they may not transfer to other GPU architectures

## Evidence

- [discussion] Expensive, per-antenna terms are computed in an EK kernel, and output to the GPU's main memory.: [evidence span withheld — anchor status: not_found]
- [discussion] These per-antenna terms are read in by a B Sum kernel, and combined to form per-baseline terms.: [evidence span withheld — anchor status: not_found]
