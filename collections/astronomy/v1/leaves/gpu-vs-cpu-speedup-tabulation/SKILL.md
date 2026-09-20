---
name: gpu-vs-cpu-speedup-tabulation
description: Use when a paper reports per-problem-size wall-clock timings for two
  or more implementations (e.g. Montblanc on an NVIDIA K40 vs OSKAR's GPU RIME on
  the same K40 vs MeqTrees on a dual hexacore Intel E5-2620v2 CPU) and the ratios
  must be tabulated and validated.
when_to_use_negative:
- The implementations were not benchmarked on the same hardware or problem sizes,
  so speedup ratios are not directly comparable
- The goal is to re-run the benchmarks empirically on new hardware rather than tabulate
  the paper's reported timings
- Only aggregate headline speedups (e.g. 'approximately 250 times faster') are needed
  without per-problem-size breakdown
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Montblanc
  role: GPU implementation of the RIME whose timings form the numerator of the speedup
    ratios; benchmarked on an NVIDIA K40
  repo: https://github.com/ska-sa/montblanc
- name: OSKAR
  role: GPU-implemented RIME simulator used as the GPU-vs-GPU baseline for single
    (7.7x) and double (12x) precision comparison
- name: MeqTrees
  role: CPU-based RIME implementation (dual hexacore Intel E5-2620v2) used as the
    CPU baseline for the ~250x speedup comparison
- name: NVIDIA K40
  role: Shared GPU platform on which Montblanc and OSKAR timings were measured
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.003
    title: 'Montblanc : GPU accelerated radio interferometer measurement equations
      in support of Bayesian inference for radio observations'
  source_content_hash: sha256:a41d48d916968caae35d6f30c73dc72fa74467c4fdd11525436e1f09bb566ed3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ska-sa/montblanc
  tools:
  - MeqTrees
  - Montblanc
  - NVIDIA K40
  - OSKAR
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# GPU vs CPU speedup tabulation

## Summary

Reproduces a benchmark timing/speedup table (e.g. Table 6 of the Montblanc paper) that compares a GPU-accelerated implementation against reference CPU and GPU baselines across problem sizes and floating-point precisions, storing per-problem timings and computed speedup ratios in a structured CSV.

## When to use

Use when a paper reports per-problem-size wall-clock timings for two or more implementations (e.g. Montblanc on an NVIDIA K40 vs OSKAR's GPU RIME on the same K40 vs MeqTrees on a dual hexacore Intel E5-2620v2 CPU) and the ratios must be tabulated and validated.

## When NOT to use

- The implementations were not benchmarked on the same hardware or problem sizes, so speedup ratios are not directly comparable
- The goal is to re-run the benchmarks empirically on new hardware rather than tabulate the paper's reported timings
- Only aggregate headline speedups (e.g. 'approximately 250 times faster') are needed without per-problem-size breakdown

## Inputs

- Published benchmark timing table (Table 6) with per-antenna-count timings for Montblanc, OSKAR and Meq Trees
- Antenna counts: 7, 14, 27, 64, 128, 192
- Precision split: single (float) and double precision floating-point timings
- Hardware specifications: NVIDIA K40 GPU and dual hexacore Intel E5-2620v2 CPU

## Outputs

- montblanc_benchmark_timings.csv with 12 problem-size/precision rows (6 antenna counts x 2 precisions)
- DataFrame columns for per-implementation timings and speedup ratios (Montblanc vs Meq Trees ~250x; Montblanc vs OSKAR 7.7x single / 12x double)

## How to apply

Read the paper's timing table (Table 6) giving per-antenna-count timings for the implementations and hardware platforms: Montblanc's RIME on an NVIDIA K40, OSKAR's GPU-implemented RIME on the same K40, and MeqTrees on a dual hexacore Intel E5-2620v2 CPU. Tabulate the timings in pandas as a DataFrame indexed by antenna count (7, 14, 27, 64, 128, 192) with separate columns for single (float) and double precision. Compute speed-up ratio columns: Montblanc vs Meq Trees (approximately 250x, e.g. 93.51 and 247.39 depending on problem size) and Montblanc vs OSKAR (7.7x for single and 12x for double precision, noting that Montblanc's advantage over OSKAR drops at large antenna counts but remains at least 7.7x and 12x). Write the DataFrame with all timing and ratio columns to montblanc_benchmark_timings.csv. The rationale is that precision (single vs double float) and problem size (antenna/baseline count) are the axes along which GPU vs CPU speedups vary, so both must be recorded per row to reproduce the reported figures.

## Related tools

- **Montblanc** (GPU implementation of the RIME whose timings form the numerator of the speedup ratios; benchmarked on an NVIDIA K40) — https://github.com/ska-sa/montblanc
- **OSKAR** (GPU-implemented RIME simulator used as the GPU-vs-GPU baseline for single (7.7x) and double (12x) precision comparison)
- **MeqTrees** (CPU-based RIME implementation (dual hexacore Intel E5-2620v2) used as the CPU baseline for the ~250x speedup comparison)
- **NVIDIA K40** (Shared GPU platform on which Montblanc and OSKAR timings were measured)

## Examples

```
import pandas as pd; df = pd.DataFrame(rows, index=pd.Index([7,14,27,64,128,192], name='nant')); df.to_csv('montblanc_benchmark_timings.csv')
```

## Evaluation signals

- Output CSV exists and contains exactly 12 rows covering the six antenna counts (7, 14, 27, 64, 128, 192) at both precisions
- Montblanc vs OSKAR ratio columns reproduce the reported values: 7.7 for single and 12 for double precision (at least these values at large antenna counts)
- Montblanc vs Meq Trees ratio column is approximately 250 (e.g. 93.51 and 247.39 depending on problem size), matching the paper's stated figure
- Timing columns are non-negative and monotonically consistent with increasing antenna count within each implementation and precision

## Limitations

- Montblanc's advantage over OSKAR drops at large antenna counts, so headline speedups should not be extrapolated to all problem sizes
- MeqTrees timings come from a different hardware platform (dual hexacore Intel E5-2620v2 CPU) than the GPU comparisons, so the ~250x figure mixes hardware differences with implementation differences
- The paper reports single-GPU results only; multi-GPU and clustered configurations were not benchmarked

## Evidence

- [abstract] Per-problem-size timings reproduce Table 6 speedups: [evidence span withheld — anchor status: not_found]
- [other] Dataset row schema from the surfacing workflow card: [evidence span withheld — anchor status: not_found]
- [other] Validation criterion from the surfacing workflow card: [evidence span withheld — anchor status: not_found]
