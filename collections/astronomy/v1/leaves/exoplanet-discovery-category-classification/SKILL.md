---
name: exoplanet-discovery-category-classification
description: Use when a radial-velocity survey data release (e.g. CARMENES DR1) provides
  planet and sample tables and the goal is to reproduce the published yield tally
  broken down by planet provenance/status category.
when_to_use_negative:
- The analysis goal is deriving radial velocities or activity indicators from spectra
  (use the CCF pipeline, e.g. raccoon, instead of classifying catalog rows).
- The input is raw CARMENES FITS spectra rather than the processed DR1 planet/sample
  tables — classification requires the tabulated planet status/provenance fields.
- Estimating planet occurrence rates (e.g. 1.44 ± 0.20 planets per star) rather than
  tallying discovery categories.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: CARMENES
  role: Source survey instrument whose DR1 planet catalog is being classified and
    tallied
- name: Python
  role: pandas-based loading, category assignment, and groupby tally of the DR1 planet
    table
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202244879
    title: The CARMENES search for exoplanets around M dwarfs
  source_content_hash: sha256:4acef4e4cec3d56b86dbee3eaba5b45b180cfbd18af67b03e6736df6a8b676cc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CARMENES
  - Python
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# exoplanet-discovery-category-classification

## Summary

Classify each planet in the CARMENES DR1 planet catalog into one of three survey-yield categories — new discoveries, re-analysed planets, and confirmed planets from transiting candidate follow-up — and tally counts against the reported breakdown (33 / 17 / 26).

## When to use

Use when a radial-velocity survey data release (e.g. CARMENES DR1) provides planet and sample tables and the goal is to reproduce the published yield tally broken down by planet provenance/status category.

## When NOT to use

- The analysis goal is deriving radial velocities or activity indicators from spectra (use the CCF pipeline, e.g. raccoon, instead of classifying catalog rows).
- The input is raw CARMENES FITS spectra rather than the processed DR1 planet/sample tables — classification requires the tabulated planet status/provenance fields.
- Estimating planet occurrence rates (e.g. 1.44 ± 0.20 planets per star) rather than tallying discovery categories.

## Inputs

- CARMENES Data Release 1 (DR1) planet table
- CARMENES DR1 sample table
- Reported category counts: 33 new planets, 17 re-analysed planets, 26 confirmed planets from transiting candidate follow-up

## Outputs

- planet_category_tally.csv with category-vs-count rows (new=33, re-analysed=17, confirmed=26)

## How to apply

Download and load the planet and sample tables from the public CARMENES DR1 release with pandas. Inspect the table columns to find the status/provenance fields that distinguish the three categories defined in the DR1 abstract: 33 new planets, 17 re-analysed planets, and 26 confirmed planets from transiting candidate follow-up. Assign each tabulated planet to exactly one of the three categories (a decision point when a planet could plausibly fall into multiple classes — resolve using the status/provenance field semantics rather than double counting). Aggregate the assignments into per-category counts with a groupby tally and write the result to planet_category_tally.csv. Validation is exact: each category count must match the published breakdown.

## Related tools

- **CARMENES** (Source survey instrument whose DR1 planet catalog is being classified and tallied)
- **Python** (pandas-based loading, category assignment, and groupby tally of the DR1 planet table)

## Examples

```
import pandas as pd; planets = pd.read_csv('carmenes_dr1_planets.csv'); tally = planets.groupby('category').size(); tally.to_csv('planet_category_tally.csv')
```

## Evaluation signals

- The tally yields exactly 33 planets in the 'new' category, matching the DR1 abstract.
- The tally yields exactly 17 planets in the 're-analysed' category and 26 in the 'confirmed from transiting candidate follow-up' category.
- Every planet in the DR1 planet table is assigned to exactly one category (no double counting, no unclassified rows).
- planet_category_tally.csv exists with a category-vs-count schema and row counts summing to the total DR1 planet population.

## Limitations

- Category assignment depends on correctly identifying the DR1 status/provenance fields; ambiguous columns can misclassify planets between 'new', 're-analysed', and 'confirmed' categories.
- Counts are valid only for the 2016–2020 GTO DR1 dataset (19,633 spectra, 362 targets); later releases or re-tabulations may change the breakdown.

## Evidence

- [abstract] Survey yield breakdown with exact counts: "including 33 new planets, 17 re-analysed planets, and 26 confirmed planets from transiting candidate follow-up"
- [abstract] DR1 dataset scope: "We present the CARMENES Data Release 1 (DR1), which makes public all observations obtained during the GTO of the CARMENES survey."
- [abstract] Dataset extent underlying the tally: "spanning from 2016 to 2020, during which 19 633 spectra for a sample of 362 targets were collected"
