---
name: radio-interferometer-performance-ratio-computation
description: Use when comparing RIME simulator performance across problem sizes (7-192
  antennas) or float precisions, or when reproducing the paper's reported speed-ups
  (~250x vs MeqTrees, 7.7x/12x vs OSKAR).
when_to_use_negative:
- The comparison requires hardware parity details (Montblanc and OSKAR both on the
  K40; MeqTrees on the E5-2620v2 CPU) that are absent from the input timings — ratios
  would be meaningless
- The goal is modelling or simulation of visibilities (e.g. BIRO χ² evaluation), not
  benchmark tabulation — use the RIME evaluation workflow instead
- Timings come from a different GPU/CPU setup than the K40/E5-2620v2; the paper's
  7.7x, 12x and ~250x figures are hardware-specific and would not validate the reproduction
tools:
- name: Montblanc
  role: Subject of the benchmark; GPU RIME implementation timed on an NVIDIA K40
  repo: https://github.com/ska-sa/montblanc
- name: OSKAR
  role: Baseline simulator whose GPU-implemented RIME components are compared against
    Montblanc on the same K40
- name: MeqTrees
  role: CPU baseline (dual hexacore Intel E5-2620v2) against which the ~250x speed-up
    is computed
- name: NVIDIA K40
  role: Shared GPU hardware used for the Montblanc and OSKAR timings
- name: Python
  role: Language used to implement Montblanc and to build the pandas tabulation of
    timings
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.003
    title: 'Montblanc : GPU accelerated radio interferometer measurement equations
      in support of Bayesian inference for radio observations'
  source_content_hash: sha256:c49039ae845e7ef0d05816c29ff76ac3d6af1d9e6522e4a3a689c90e824dcd58
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ska-sa/montblanc
  tools:
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
yes_rate: 0.8888888888888888
n_claims: 9
---

# radio-interferometer-performance-ratio-computation

## Summary

Reproduces and extends the benchmark timing comparison from the Montblanc paper (Table 6), tabulating RIME computation times for Montblanc on an NVIDIA K40, OSKAR's GPU RIME on the same K40, and MeqTrees on a dual hexacore Intel E5-2620v2 CPU, then computing speed-up ratios across antenna counts and floating-point precision.

## When to use

Use when comparing RIME simulator performance across problem sizes (7-192 antennas) or float precisions, or when reproducing the paper's reported speed-ups (~250x vs MeqTrees, 7.7x/12x vs OSKAR).

## When NOT to use

- The comparison requires hardware parity details (Montblanc and OSKAR both on the K40; MeqTrees on the E5-2620v2 CPU) that are absent from the input timings — ratios would be meaningless
- The goal is modelling or simulation of visibilities (e.g. BIRO χ² evaluation), not benchmark tabulation — use the RIME evaluation workflow instead
- Timings come from a different GPU/CPU setup than the K40/E5-2620v2; the paper's 7.7x, 12x and ~250x figures are hardware-specific and would not validate the reproduction

## Inputs

- Per-problem-size RIME computation timings from the paper's Table 6 (Montblanc K40, OSKAR K40, MeqTrees E5-2620v2)
- Antenna counts: 7, 14, 27, 64, 128, 192
- Floating-point precision labels (single/float and double)

## Outputs

- pandas DataFrame of timings indexed by antenna count with single/double precision columns
- Speed-up ratio columns: Montblanc vs MeqTrees (~250x) and Montblanc vs OSKAR (7.7x single, 12x double)
- montblanc_benchmark_timings.csv with 12 problem-size/precision rows

## How to apply

Read the per-antenna-count timing data (Table 6 of Perkins et al. 2015) for Montblanc's RIME on an NVIDIA K40, OSKAR's GPU-implemented RIME on the same K40, and MeqTrees on a dual hexacore Intel E5-2620v2 CPU. Tabulate the timings in pandas as a structured DataFrame indexed by antenna count (7, 14, 27, 64, 128, 192) with separate columns for single (float) and double precision. Compute ratio columns: Montblanc vs MeqTrees (approximately 250x, e.g. 93.51 and 247.39 depending on problem size) and Montblanc vs OSKAR (7.7x single precision, 12x double precision). Write the full DataFrame to montblanc_benchmark_timings.csv. Rationale: the ratios quantify how Montblanc's GPU advantage varies with problem size, since its edge over OSKAR narrows at large antenna counts (remaining at least 7.7x and 12x).

## Related tools

- **Montblanc** (Subject of the benchmark; GPU RIME implementation timed on an NVIDIA K40) — https://github.com/ska-sa/montblanc
- **OSKAR** (Baseline simulator whose GPU-implemented RIME components are compared against Montblanc on the same K40)
- **MeqTrees** (CPU baseline (dual hexacore Intel E5-2620v2) against which the ~250x speed-up is computed)
- **NVIDIA K40** (Shared GPU hardware used for the Montblanc and OSKAR timings)
- **Python** (Language used to implement Montblanc and to build the pandas tabulation of timings)

## Examples

```
import pandas as pd; df.to_csv('montblanc_benchmark_timings.csv', index=False)  # after tabulating Table 6 timings and computing the 7.7x/12x/~250x ratio columns
```

## Evaluation signals

- Output CSV exists and contains exactly 12 rows (6 antenna counts x 2 precisions)
- Montblanc-vs-OSKAR ratio columns reproduce the reported 7.7 (single) and 12 (double) figures
- Montblanc-vs-MeqTrees ratio column falls near the reported ~250x (e.g. 93.51 to 247.39 across problem sizes)
- Montblanc's advantage over OSKAR is monotonically weaker as antenna count increases toward 192, staying at least 7.7x/12x
- Each timing row is tagged with its precision (single vs double) and hardware (K40 for Montblanc/OSKAR, E5-2620v2 for MeqTrees)

## Limitations

- MeqTrees runs on CPU (dual hexacore Intel E5-2620v2) while Montblanc and OSKAR run on the same K40 GPU, so the ~250x figure is a heterogeneous comparison
- The 7.7x and 12x figures are worst-case bounds at large antenna counts; the advantage at smaller problem sizes is larger and varies with problem size
- Montblanc (as published) supports only single GPUs and substitutes analytic expressions for E_ps (DDE) terms, which affects comparability of the benchmarked workloads

## Evidence
