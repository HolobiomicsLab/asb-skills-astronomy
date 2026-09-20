---
name: scientific-table-structuring
description: Use when a paper reports per-problem-size performance numbers (timings,
  speed-ups across precisions or configurations) that must be tabulated, computed
  into ratio columns, and exported for downstream benchmark analysis.
when_to_use_negative:
- The source only gives a single aggregate speed-up figure with no per-problem-size
  breakdown to structure
- The goal is to re-run the benchmarks themselves rather than tabulate and validate
  already-published numbers
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Montblanc
  role: Subject of the benchmark; its RIME computation times on the NVIDIA K40 form
    the timing columns being structured
  repo: https://github.com/ska-sa/montblanc
- name: OSKAR
  role: GPU-implemented RIME comparison baseline on the same K40; timings feed the
    Montblanc-vs-OSKAR speed-up ratio column
- name: MeqTrees
  role: CPU comparison baseline (dual hexacore Intel E5-2620v2); timings feed the
    ~250x speed-up ratio column
- name: Python
  role: Language used to tabulate the timings in pandas and write the output CSV
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.06.003
    title: 'Montblanc : GPU accelerated radio interferometer measurement equations
      in support of Bayesian inference for radio observations'
  source_content_hash: sha256:b1f735e8d973aed8b4ff8fb0a8fe759534a9ceb5f83351431518ccc5ed341229
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# scientific-table-structuring

## Summary

Reproduce and structure benchmark timing tables from a paper (e.g. Montblanc's Table 6) into a tidy DataFrame/CSV indexed by problem size with per-precision timing and speed-up ratio columns, enabling validation against reported figures.

## When to use

Use when a paper reports per-problem-size performance numbers (timings, speed-ups across precisions or configurations) that must be tabulated, computed into ratio columns, and exported for downstream benchmark analysis.

## When NOT to use

- The source only gives a single aggregate speed-up figure with no per-problem-size breakdown to structure
- The goal is to re-run the benchmarks themselves rather than tabulate and validate already-published numbers

## Inputs

- Published benchmark timing table (Table 6) with per-antenna-count RIME timings for Montblanc (NVIDIA K40), OSKAR (K40 GPU), and MeqTrees (dual hexacore Intel E5-2620v2 CPU)
- Antenna counts: 7, 14, 27, 64, 128, 192
- Floating-point precision labels: single (float) and double

## Outputs

- pandas DataFrame indexed by antenna count with timing columns per precision and speed-up ratio columns (Montblanc vs MeqTrees, Montblanc vs OSKAR)
- montblanc_benchmark_timings.csv with 12 problem-size/precision rows (6 antenna counts x 2 precisions)

## How to apply

Read the paper's timing table (here Table 6 of the Montblanc paper, DOI 10.1016/j.ascom.2015.06.003), which gives RIME timings for Montblanc on an NVIDIA K40, OSKAR's GPU-implemented RIME on the same K40, and MeqTrees on a dual hexacore Intel E5-2620v2 CPU, per antenna count (7, 14, 27, 64, 128, 192). Load these into a pandas DataFrame indexed by antenna count with separate columns for single (float) and double precision. Compute derived ratio columns: Montblanc vs MeqTrees speed-up (~250x, e.g. 93.51 and 247.39 depending on problem size) and Montblanc vs OSKAR (7.7x single precision, 12x double precision). Write the result to a CSV (e.g. montblanc_benchmark_timings.csv) containing the 12 problem-size/precision rows. Validate that the computed ratios reproduce the paper's stated figures (7.7, 12, ~250) before treating the table as trustworthy.

## Related tools

- **Montblanc** (Subject of the benchmark; its RIME computation times on the NVIDIA K40 form the timing columns being structured) — https://github.com/ska-sa/montblanc
- **OSKAR** (GPU-implemented RIME comparison baseline on the same K40; timings feed the Montblanc-vs-OSKAR speed-up ratio column)
- **MeqTrees** (CPU comparison baseline (dual hexacore Intel E5-2620v2); timings feed the ~250x speed-up ratio column)
- **Python** (Language used to tabulate the timings in pandas and write the output CSV)

## Examples

```
df = pd.read_csv('table6_timings.csv').set_index('nant'); df['speedup_meqtrees'] = df.meqtrees_time / df.montblanc_time; df['speedup_oskar'] = df.oskar_time / df.montblanc_time; df.to_csv('montblanc_benchmark_timings.csv')
```

## Evaluation signals

- Output CSV exists and contains exactly 12 rows (6 antenna counts x 2 precisions)
- Montblanc vs OSKAR ratio columns reproduce the reported 7.7 (single precision) and 12 (double precision)
- Montblanc vs MeqTrees ratio column falls around ~250x, consistent with values like 93.51 and 247.39 depending on problem size
- DataFrame index covers exactly the antenna counts 7, 14, 27, 64, 128, 192
- Computed ratios match the paper's stated figures within tolerance

## Limitations

- Montblanc's advantage over OSKAR shrinks at large antenna counts (it remains 'at least' 7.7x/12x faster), so ratio values vary with problem size and are lower bounds at large antenna counts
- The comparison mixes hardware platforms: Montblanc and OSKAR timings are on an NVIDIA K40 GPU while MeqTrees runs on a dual hexacore Intel E5-2620v2 CPU, so cross-tool ratios are not hardware-controlled
- Montblanc supports only single GPUs; multi-GPU and clustered timings are not covered by the table

## Evidence

- [abstract] On an NVIDIA K40, it is approximately 250 times faster than Meq Trees on a dual hexacore Intel E5-2620v2 CPU: [evidence span withheld — anchor status: not_found]
- [results] Per-problem-size timings in Table 6 show Montblanc's advantage over OSKAR drops at large antenna counts: [evidence span withheld — anchor status: not_found]
