---
name: co-flux-benchmark-comparison
description: Use when a fast semi-analytical CO flux model must be validated against
  thermochemical reference fluxes, and you have paired model outputs on matching disc
  structural parameters plus a paper-stated tolerance (factor of three).
when_to_use_negative:
- When the goal is to model CO fluxes for hundreds of discs in a population study
  — that is what the semi-analytical method itself is for; this skill only validates
  it against DALI.
- When inputs are observational data (Lupus or Upper Sco ALMA measurements) rather
  than model-vs-model flux pairs — the benchmark criterion is defined for semi-analytical
  vs DALI fluxes.
- When no thermochemical reference fluxes or matching disc structural parameters are
  available, since pairing on disc structural parameters is a prerequisite.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: DALI
  role: Thermochemical code whose reference fluxes supply the benchmark against which
    semi-analytical 12CO fluxes are compared
- name: COpops
  role: Repository hosting the DALI thermochemical reference fluxes and the semi-analytical
    12CO flux modelling code
  repo: github.com/fzagaria/COpops
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202346164
    title: Testing protoplanetary disc evolution with CO fluxes
  source_content_hash: sha256:d6711d378f375000c1ef7cc617c9174cbd341c5f2f06b845bf6d64cf47b3fd61
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COpops
  - DALI
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# co-flux-benchmark-comparison

## Summary

Benchmarks a semi-analytical 12CO disc flux model against DALI thermochemical reference models for the DALI-benchmark disc set, validating agreement via per-model flux ratios within a factor of three.

## When to use

Use when a fast semi-analytical CO flux model must be validated against thermochemical reference fluxes, and you have paired model outputs on matching disc structural parameters plus a paper-stated tolerance (factor of three).

## When NOT to use

- When the goal is to model CO fluxes for hundreds of discs in a population study — that is what the semi-analytical method itself is for; this skill only validates it against DALI.
- When inputs are observational data (Lupus or Upper Sco ALMA measurements) rather than model-vs-model flux pairs — the benchmark criterion is defined for semi-analytical vs DALI fluxes.
- When no thermochemical reference fluxes or matching disc structural parameters are available, since pairing on disc structural parameters is a prerequisite.

## Inputs

- Semi-analytical 12CO flux table produced by the upstream flux-modelling step (e.g. task_001/flux_table, readable with pandas)
- DALI thermochemical reference fluxes for the DALI-benchmark disc set, deposited with the COpops repository (github.com/fzagaria/COpops)
- Disc structural parameter metadata used to pair semi-analytical models with their DALI counterparts

## Outputs

- Per-model ratio table written to dali_benchmark_ratios.csv with columns: model identifier, semi-analytical flux, DALI flux, flux ratio, within-factor-of-three pass/fail flag

## How to apply

Load the semi-analytical 12CO flux table (e.g. task_001/flux_table) with pandas, and load the DALI thermochemical reference fluxes deposited with the COpops repository (github.com/fzagaria/COpops) for the DALI-benchmark disc set. Match each semi-analytical model to its corresponding DALI thermochemical model on disc structural parameters, then compute the per-model ratio F(12CO, semi-analytical) / F(12CO, DALI). Apply the paper's benchmark criterion that each ratio fall within a factor of three (0.333 <= ratio <= 3.0) and record a pass/fail flag per model. Write the per-model table — model identifier, semi-analytical flux, DALI flux, ratio, within-factor-of-three flag — to dali_benchmark_ratios.csv. The rationale is that the paper benchmarked its semi-analytical method 'against the results of DALI thermochemical models, recovering an agreement within a factor of three', so this ratio band is the validated accuracy envelope for the fast method.

## Related tools

- **DALI** (Thermochemical code whose reference fluxes supply the benchmark against which semi-analytical 12CO fluxes are compared)
- **COpops** (Repository hosting the DALI thermochemical reference fluxes and the semi-analytical 12CO flux modelling code) — github.com/fzagaria/COpops

## Examples

```
import pandas as pd; df = pd.read_csv('task_001/flux_table'); ratios = df['F_sa_12CO'] / df['F_dali_12CO']; df['within_factor_of_three'] = ratios.between(1/3, 3.0); df.to_csv('dali_benchmark_ratios.csv', index=False)
```

## Evaluation signals

- Every computed flux ratio satisfies 0.333 <= F(12CO, semi-analytical)/F(12CO, DALI) <= 3.0, mirroring the paper's 'agreement within a factor of three'
- dali_benchmark_ratios.csv contains one row per benchmark model with identifier, both fluxes, ratio, and pass/fail flag populated
- Each semi-analytical model is matched to exactly one DALI thermochemical model on disc structural parameters (no unmatched or duplicated pairs)
- Flux ratios are computed consistently with the same model set as the COpops DALI-benchmark deposits
- Re-deriving ratios from the two flux columns of the CSV reproduces the recorded ratio column exactly

## Limitations

- The factor-of-three agreement is a loose tolerance; individual models can deviate by up to 3x and still 'pass', so per-model ratios should be inspected, not just the aggregate flag
- Agreement is established only for the DALI-benchmark disc condition set; extrapolation to other disc structures is not certified by this benchmark
- The paper notes that uncertainties on initial conditions limited interpretation of observations, so benchmark agreement does not resolve underlying disc evolution mechanisms (viscosity vs MHD winds)

## Evidence

- [abstract] benchmarked it against the results of DALI thermochemical models, recovering an agreement within a factor of three: "benchmarked it against the results of DALI thermochemical models, recovering an agreement within a factor of three"
- [readme] running computationally-expensive thermochemical models for hundreds of discs can be prohibitively time-consuming: "running computationally-expensive thermochemical models for hundreds of discs can be prohibitively time-consuming"
- [readme] compared to more-complex physico-chemical models, we recover a good agreement within a factor of three: "compared to more-complex physico-chemical models, we recover a good agreement within a factor of three"
