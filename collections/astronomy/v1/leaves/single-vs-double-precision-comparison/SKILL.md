---
name: single-vs-double-precision-comparison
description: Use when benchmarking a GPU kernel pipeline (e.g. Montblanc's EK and
  B Sum kernels on a K40) and you need to quantify how float vs double precision changes
  runtime, occupancy, and speed-ups against reference implementations.
when_to_use_negative:
- Your workload includes evaluation of G_ps (DIE) matrices or full E_ps (DDE) matrix
  evaluation — Montblanc substitutes analytic expressions and omits G_ps, so cross-precision
  speed-ups will not transfer
- You need multi-GPU or distributed benchmarking — Montblanc supports only single
  GPUs, so K40-based precision comparisons cannot be scaled out
- Your problem exceeds GPU memory budget without subdividing along the time dimension
  — timings would conflate subdivision overhead with precision effects
tools:
- name: Montblanc
  role: GPU RIME implementation whose float/double timings and kernel occupancies
    are compared
  repo: https://github.com/ska-sa/montblanc
- name: OSKAR
  role: reference GPU-implemented RIME simulator used for the 7.7x (float) and 12x
    (double) speed-up comparison on the same K40
- name: MeqTrees
  role: CPU-based reference simulator (dual hexacore Intel E5-2620v2) for the ~250x
    speed-up baseline
- name: NVIDIA K40
  role: shared GPU hardware on which both-precision benchmarks and profiling are run
- name: CUDA
  role: architecture Montblanc is implemented on; NVIDIA Profiler supplies registers/occupancy
    metrics per kernel
- name: Python
  role: language used to tabulate timings in pandas and serialize CSV/JSON outputs
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.003
    title: 'Montblanc : GPU accelerated radio interferometer measurement equations
      in support of Bayesian inference for radio observations'
  source_content_hash: sha256:73f68d7977b8724b6a49dee30a2f71b02baa11f211878780fc115a77fa563d8e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ska-sa/montblanc
  tools:
  - CUDA
  - MeqTrees
  - Montblanc
  - NVIDIA K40
  - OSKAR
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# single-vs-double-precision-comparison

## Summary

Compares single (float32) and double (float64) precision performance of a GPU RIME implementation by tabulating per-antenna-count timings, speed-up ratios versus other simulators, and NVIDIA Profiler kernel metrics (registers, threads/block, occupancy) for both precisions.

## When to use

Use when benchmarking a GPU kernel pipeline (e.g. Montblanc's EK and B Sum kernels on a K40) and you need to quantify how float vs double precision changes runtime, occupancy, and speed-ups against reference implementations.

## When NOT to use

- Your workload includes evaluation of G_ps (DIE) matrices or full E_ps (DDE) matrix evaluation — Montblanc substitutes analytic expressions and omits G_ps, so cross-precision speed-ups will not transfer
- You need multi-GPU or distributed benchmarking — Montblanc supports only single GPUs, so K40-based precision comparisons cannot be scaled out
- Your problem exceeds GPU memory budget without subdividing along the time dimension — timings would conflate subdivision overhead with precision effects

## Inputs

- Per-problem-size timing table (Table 6): Montblanc K40 timings, OSKAR GPU RIME timings, MeqTrees CPU timings for six antenna counts (7, 14, 27, 64, 128, 192) x two precisions
- NVIDIA Profiler kernel metrics (Tables 3-5): registers per thread, threads per block, occupancy, compute/memory instruction split, memory-hierarchy bandwidths (shared, global, L1, L2, DRAM) for EK and B Sum kernels, float and double

## Outputs

- montblanc_benchmark_timings.csv with 12 problem-size/precision rows and speed-up ratio columns (vs OSKAR: 7.7 float / 12 double; vs MeqTrees: ~250x)
- JSON records, one per kernel per precision, holding registers, threads/block, occupancy, and bandwidth values

## How to apply

Run the same RIME workload at both precisions on identical hardware (NVIDIA K40 for Montblanc and OSKAR's GPU RIME; dual hexacore Intel E5-2620v2 CPU for MeqTrees), sweeping antenna counts (7, 14, 27, 64, 128, 192). Tabulate per-problem-size timings from the paper's Table 6 in a pandas DataFrame indexed by antenna count with separate single/double precision columns, then compute ratio columns: Montblanc vs Meq Trees (~250x, e.g. 93.51 and 247.39 depending on problem size) and Montblanc vs OSKAR (7.7x float, 12x double at large antenna counts). Complement the timing comparison with NVIDIA Profiler data per kernel and precision: EK kernel 31 registers/512 threads/87% occupancy (float) vs 44/256/59% (double); B Sum kernel 46/256/60% (float) vs 63/128/50% (double). Serialize timings to CSV and profiler records to JSON, one record per kernel per precision, then validate that computed ratios reproduce the paper's reported figures.

## Related tools

- **Montblanc** (GPU RIME implementation whose float/double timings and kernel occupancies are compared) — https://github.com/ska-sa/montblanc
- **OSKAR** (reference GPU-implemented RIME simulator used for the 7.7x (float) and 12x (double) speed-up comparison on the same K40)
- **MeqTrees** (CPU-based reference simulator (dual hexacore Intel E5-2620v2) for the ~250x speed-up baseline)
- **NVIDIA K40** (shared GPU hardware on which both-precision benchmarks and profiling are run)
- **CUDA** (architecture Montblanc is implemented on; NVIDIA Profiler supplies registers/occupancy metrics per kernel)
- **Python** (language used to tabulate timings in pandas and serialize CSV/JSON outputs)

## Examples

```
df.to_csv("montblanc_benchmark_timings.csv")  # after tabulating per-antenna-count float/double timings and 7.7/12/~250x ratio columns in pandas
```

## Evaluation signals

- CSV contains exactly 12 problem-size/precision rows (6 antenna counts x 2 precisions)
- Montblanc-vs-OSKAR ratio columns reproduce 7.7 (single) and 12 (double) within the paper's stated figures at large antenna counts
- Montblanc-vs-MeqTrees ratios fall near ~250x (e.g. 93.51 and 247.39 depending on problem size)
- Profiler JSON includes both kernels (EK and B Sum) and both precisions, with EK float showing 31 registers/512 threads/87% occupancy and double 44/256/59%
- Output CSV file existence and schema (antenna-count index plus float/double timing and ratio columns) verified programmatically

## Limitations

- The kernel is theoretically memory bound, but profiling shows a balanced compute/memory split due to L1/L2 cache retention, so bandwidth-based predictions of precision scaling may not match measured timings
- G_ps (DIE) matrices are not evaluated and E_ps (DDE) terms use analytic expressions, limiting scope of the compared workload
- Only single GPUs are supported; results do not extend to multi-GPU or clustered configurations, and out-of-core/distributed data-transfer bottlenecks remain unsolved

## Evidence

- [discussion] EK/B Sum kernel split that the per-precision profiler metrics describe: [evidence span withheld — anchor status: not_found]
