---
name: planet-sample-tally-aggregation
description: Use when a task requires reproducing an exoplanet yield tally from a
  published survey's planet tables, assigning each catalogued planet to exactly one
  provenance/status category and validating counts against reported numbers.
when_to_use_negative:
- The goal is computing occurrence rates (e.g. the survey's 1.44 ± 0.20 planets per
  star) rather than a provenance tally — that requires the full survey sample and
  detection completeness, not category assignment.
- The input is raw spectra or RV time series (e.g. the 19,633 CARMENES spectra or
  18,642 RVs) rather than an already-tabulated planet catalog — no status/provenance
  fields exist yet to classify.
- Planet candidates come from a different survey whose status vocabulary does not
  map to the CARMENES new/re-analysed/confirmed categories without a documented crosswalk.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Python
  role: pandas-based loading, status-field inspection, one-of-three category assignment,
    groupby tally, and CSV export of the DR1 planet tables
  repo: github.com/mlafarga/raccoon
- name: CARMENES
  role: source instrument whose GTO Data Release 1 supplies the planet and sample
    tables being tallied
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202244879
    title: The CARMENES search for exoplanets around M dwarfs
  source_content_hash: sha256:2f491ab69ea7a8449e1bcf7ebcbb8ce49ba216f47140b4507417d6b124ebc56b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CARMENES
  - Python
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# planet-sample-tally-aggregation

## Summary

Classify and tally the planets tabulated in the CARMENES DR1 planet catalog into the survey's reported provenance categories — 33 new planets, 17 re-analysed planets, and 26 confirmed planets from transiting candidate follow-up — using pandas groupby aggregation validated against the published breakdown.

## When to use

Use when a task requires reproducing an exoplanet yield tally from a published survey's planet tables, assigning each catalogued planet to exactly one provenance/status category and validating counts against reported numbers.

## When NOT to use

- The goal is computing occurrence rates (e.g. the survey's 1.44 ± 0.20 planets per star) rather than a provenance tally — that requires the full survey sample and detection completeness, not category assignment.
- The input is raw spectra or RV time series (e.g. the 19,633 CARMENES spectra or 18,642 RVs) rather than an already-tabulated planet catalog — no status/provenance fields exist yet to classify.
- Planet candidates come from a different survey whose status vocabulary does not map to the CARMENES new/re-analysed/confirmed categories without a documented crosswalk.

## Inputs

- CARMENES DR1 planet catalog table (public release, loaded with pandas)
- CARMENES DR1 sample table (public release, loaded with pandas)
- Reported target breakdown: 33 new planets, 17 re-analysed planets, 26 confirmed planets from transiting candidate follow-up

## Outputs

- planet_category_tally.csv: category-vs-count table with three rows (new, re-analysed, confirmed transiting follow-up)

## How to apply

Download and load the planet and sample tables from the public CARMENES Data Release 1 (DR1) with pandas. Inspect the table columns to identify the planet status/provenance fields that distinguish the three reported categories: new discoveries, re-analysed planets, and confirmed planets from transiting candidate follow-up. Assign each tabulated planet to exactly one of the three categories so the classification is mutually exclusive and exhaustive. Aggregate with a pandas groupby tally and write the category-vs-count table to planet_category_tally.csv. Validate that the three counts equal the reported breakdown of 33 new planets, 17 re-analysed planets, and 26 confirmed planets from transiting candidate follow-up; any mismatch indicates a misread status field or planets assigned to multiple categories.

## Related tools

- **Python** (pandas-based loading, status-field inspection, one-of-three category assignment, groupby tally, and CSV export of the DR1 planet tables) — github.com/mlafarga/raccoon
- **CARMENES** (source instrument whose GTO Data Release 1 supplies the planet and sample tables being tallied)

## Examples

```
df = pd.read_csv('carmenes_dr1_planets.csv'); df.groupby('status').size().to_csv('planet_category_tally.csv')
```

## Evaluation signals

- The three category counts equal exactly 33 new planets, 17 re-analysed planets, and 26 confirmed planets from transiting candidate follow-up.
- Every planet in the DR1 planet tables appears in exactly one category (no double assignment, no unclassified rows).
- planet_category_tally.csv exists and contains a category-vs-count mapping with three rows summing to the total DR1 planet yield.
- Column inspection documentation traces each category to an explicit status/provenance field value in the DR1 tables.

## Limitations

- Validation is anchored to the DR1 abstract figures (33/17/26); later data releases or re-releases of the DR1 tables may shift counts and break the invariant.
- The tally depends on correctly interpreting the status/provenance fields in the DR1 tables; ambiguous or multi-valued status entries would need a documented tie-breaking rule.
- This skill tallies provenance categories only; it does not address the survey's occurrence-rate analysis (1.44 ± 0.20 planets per star for 1–1000 M_Earth and 1–1000 days).

## Evidence

- [abstract] Survey revealed 33 new planets, 17 re-analysed planets, and 26 confirmed planets from transiting candidate follow-up: "including 33 new planets, 17 re-analysed planets, and 26 confirmed planets from transiting candidate follow-up"
- [abstract] DR1 public release of GTO observations: "We present the CARMENES Data Release 1 (DR1), which makes public all observations obtained during the GTO of the CARMENES survey."
- [abstract] Pipeline-processed data with derived high-level products: "The data were pipeline-processed, and high-level data products, including 18 642 precise RVs for 345 targets, were derived."
