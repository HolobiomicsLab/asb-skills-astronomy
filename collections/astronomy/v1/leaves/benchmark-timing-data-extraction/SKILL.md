---
name: benchmark-timing-data-extraction
description: Use when a paper reports a timing/benchmark table comparing implementations
  across problem sizes or precision settings, and the numbers must be extracted, structured,
  and validated against reported speedup figures.
when_to_use_negative:
- The timings come from your own re-run benchmarks rather than a published table —
  measure directly instead of transcribing reported values.
- The paper reports only aggregate speedup statements (e.g. an abstract-level 'approximately
  250 times faster') with no per-problem-size breakdown to tabulate.
- 'The comparison hardware/software stacks differ from the target system, making the
  extracted ratios non-transferable (here: K40 GPU vs dual hexacore Intel E5-2620v2
  CPU).'
edam_operation: http://edamontology.org/operation_2928
tools:
- name: Montblanc
  role: GPU RIME implementation whose benchmark timings are the subject of the table
  repo: https://github.com/ska-sa/montblanc
- name: OSKAR
  role: Comparison simulator whose GPU-implemented RIME components are timed on the
    same K40 for the speedup baseline
- name: MeqTrees
  role: CPU-based comparison implementation (dual hexacore Intel E5-2620v2) providing
    the ~250x speedup baseline
- name: NVIDIA K40
  role: Shared GPU hardware on which the Montblanc and OSKAR timings were measured
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.003
    title: 'Montblanc : GPU accelerated radio interferometer measurement equations
      in support of Bayesian inference for radio observations'
  source_content_hash: sha256:6527199e82ca03d6cbb888ae294a2ef3e032d0d25c6a894ee15f1cf0263022be
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/ska-sa/montblanc
  tools:
  - MeqTrees
  - Montblanc
  - NVIDIA K40
  - OSKAR
  edam_operation: http://edamontology.org/operation_2928
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# benchmark-timing-data-extraction

## Summary

Extracts per-problem-size hardware benchmark timings from a publication's results table (e.g. Table 6 of the Montblanc paper), tabulates them in a structured pandas DataFrame, and derives speedup-ratio columns to reproduce the paper's reported performance comparisons.

## When to use

Use when a paper reports a timing/benchmark table comparing implementations across problem sizes or precision settings, and the numbers must be extracted, structured, and validated against reported speedup figures.

## When NOT to use

- The timings come from your own re-run benchmarks rather than a published table — measure directly instead of transcribing reported values.
- The paper reports only aggregate speedup statements (e.g. an abstract-level 'approximately 250 times faster') with no per-problem-size breakdown to tabulate.
- The comparison hardware/software stacks differ from the target system, making the extracted ratios non-transferable (here: K40 GPU vs dual hexacore Intel E5-2620v2 CPU).

## Inputs

- Published benchmark timing table (Table 6 of the Montblanc paper) with per-antenna-count timings for Montblanc, OSKAR, and Meq Trees
- Problem-size axis: number of antennas/baselines (7, 14, 27, 64, 128, 192)
- Floating-point precision setting (single/float vs double)
- Reported speedup figures used as anchors (7.7, 12, ~250)

## Outputs

- pandas DataFrame indexed by antenna count with per-precision timing columns
- Computed speedup-ratio columns (Montblanc vs OSKAR; Montblanc vs Meq Trees)
- montblanc_benchmark_timings.csv containing all timing and ratio columns (12 problem-size/precision rows)

## How to apply

Read the paper's benchmark table (here Table 6: Montblanc RIME timings on an NVIDIA K40, OSKAR's GPU-implemented RIME on the same K40, and Meq Trees on a dual hexacore Intel E5-2620v2 CPU) for each antenna count (7, 14, 27, 64, 128, 192). Tabulate the timings in pandas as a DataFrame indexed by antenna count with separate columns for single (float) and double precision. Compute speedup ratio columns from the raw timings, anchoring them to the paper's stated values: Montblanc vs Meq Trees approximately 250x (e.g. 93.51 and 247.39 depending on problem size), Montblanc vs OSKAR 7.7x for single and 12x for double precision. Write all timing and ratio columns to montblanc_benchmark_timings.csv, then validate the row count (12 problem-size/precision rows for six antenna counts) and that ratio columns reproduce the reported figures within the paper's stated values.

## Related tools

- **Montblanc** (GPU RIME implementation whose benchmark timings are the subject of the table) — https://github.com/ska-sa/montblanc
- **OSKAR** (Comparison simulator whose GPU-implemented RIME components are timed on the same K40 for the speedup baseline)
- **MeqTrees** (CPU-based comparison implementation (dual hexacore Intel E5-2620v2) providing the ~250x speedup baseline)
- **NVIDIA K40** (Shared GPU hardware on which the Montblanc and OSKAR timings were measured)

## Examples

```
import pandas as pd; df = pd.DataFrame(timings, index=[7,14,27,64,128,192], columns=['montblanc_float','oskar_float','meqtrees_float','montblanc_double','oskar_double','meqtrees_double']); df.to_csv('montblanc_benchmark_timings.csv')
```

## Evaluation signals

- Output CSV montblanc_benchmark_timings.csv exists and is parseable
- CSV contains exactly 12 problem-size/precision rows covering the six antenna counts (7, 14, 27, 64, 128, 192) in two precisions
- Montblanc vs OSKAR ratio columns reproduce 7.7 (single precision) and 12 (double precision) within the paper's stated figures
- Montblanc vs Meq Trees ratio columns fall around the reported ~250x range (e.g. 93.51 and 247.39 depending on problem size)
- At large antenna counts the Montblanc-vs-OSKAR advantage decreases but stays at least 7.7x and 12x for single and double precision respectively

## Limitations

- Montblanc's advantage over OSKAR shrinks as antenna counts grow; the reported minimums (7.7x single, 12x double) apply at the largest problem sizes, not uniformly.
- The ~250x Meq Trees comparison is cross-hardware (NVIDIA K40 GPU vs dual hexacore Intel E5-2620v2 CPU) and does not isolate the algorithmic contribution.
- Only single-GPU Montblanc results exist in this paper; multi-GPU and clustered configurations are not covered by these timings.

## Evidence

- [abstract] Montblanc ~250x faster than Meq Trees on K40 vs CPU: "On an NVIDIA K40, it is approximately 250 times faster than Meq Trees on a dual hexacore Intel E5–2620v2 CPU."
- [abstract] Montblanc 7.7x and 12x faster than OSKAR on same K40: "Compared to the OSKAR simulator’s GPU-implemented RIME components it is 7.7 and 12 times faster on the same K40 for single and double-precision floating point respectively."
- [intro] Meq Trees used for prior BIRO implementations, motivating the timing comparison: "Existing BIRO implementations Lochner et al. (2015) use Meq Trees (Noordam and Smirnov, 2010) to evaluate the RIME"
