---
name: data-release-product-retrieval
description: Use when the task requires enumerating spectra, targets, or derived RV/activity
  products from a public survey data release (e.g. CARMENES DR1 GTO 2016-2020) and
  validating counts against published values such as 19,633 spectra / 362 targets.
when_to_use_negative:
- When the goal is to recompute CCFs, radial velocities, or activity indicators from
  raw FITS spectra — use the raccoon CCF pipeline instead of just enumerating released
  products
- When the data are ad-hoc or unversioned observations not part of a public data release
  with a published inventory to validate against
edam_operation: http://edamontology.org/operation_2422
tools:
- name: CARMENES
  role: Spectrograph and survey whose DR1 public release supplies the spectra, RV
    products, and planet tables being retrieved and tallied
- name: Python
  role: pandas-based loading, grouping by stellar target, and groupby tallying of
    DR1 tables into summary counts
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202244879
    title: The CARMENES search for exoplanets around M dwarfs
  source_content_hash: sha256:4878b65979e28935bd52fbab48d5feecca5c61f7cb81b0449df11b57c5e9f3cd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CARMENES
  - Python
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# data-release-product-retrieval

## Summary

Retrieve and enumerate the data products of a versioned observatory data release (CARMENES Data Release 1), tallying spectra, targets, and derived high-level products (precise RV time series) against the published inventory.

## When to use

Use when the task requires enumerating spectra, targets, or derived RV/activity products from a public survey data release (e.g. CARMENES DR1 GTO 2016-2020) and validating counts against published values such as 19,633 spectra / 362 targets.

## When NOT to use

- When the goal is to recompute CCFs, radial velocities, or activity indicators from raw FITS spectra — use the raccoon CCF pipeline instead of just enumerating released products
- When the data are ad-hoc or unversioned observations not part of a public data release with a published inventory to validate against

## Inputs

- CARMENES Data Release 1 (DR1) metadata tables (spectra inventory)
- DR1 high-level data products: radial-velocity time series
- DR1 planet and sample tables with planet status/provenance fields

## Outputs

- Summary table of four counts: spectra, unique targets, precise RVs, unique RV targets
- planet_category_tally.csv: per-category counts of new / re-analysed / confirmed planets

## How to apply

Access the public CARMENES DR1 release, which contains all GTO observations (2016-2020) including raw, pipeline-processed, and high-level products. Load the DR1 metadata tables in Python (pandas) and enumerate individual spectra, then group by stellar target to count unique targets (expected: 19,633 spectra for 362 targets). Separately load the high-level radial-velocity time series and count precise RVs and their unique targets (expected: 18,642 RVs for 345 targets). For planet-product retrieval, inspect the DR1 planet and sample tables' status/provenance columns, assign each tabulated planet to exactly one category (new, re-analysed, or confirmed from transiting-candidate follow-up), and aggregate with a groupby tally before writing the counts to CSV. Rationale: the release is the authoritative, complete GTO archive, so any inventory recount must reconcile exactly with the published abstract-level breakdown.

## Related tools

- **CARMENES** (Spectrograph and survey whose DR1 public release supplies the spectra, RV products, and planet tables being retrieved and tallied)
- **Python** (pandas-based loading, grouping by stellar target, and groupby tallying of DR1 tables into summary counts)

## Examples

```
import pandas as pd; dr1 = pd.read_csv('carmenes_dr1_planets.csv'); dr1.groupby('status').size().to_csv('planet_category_tally.csv')
```

## Evaluation signals

- Spectra inventory count equals 19,633 with 362 unique targets
- High-level product count equals 18,642 precise RVs for 345 targets
- Planet category tally equals the reported breakdown: 33 new, 17 re-analysed, 26 confirmed from transiting-candidate follow-up
- Every tabulated planet is assigned to exactly one of the three categories (no double counting, no unclassified rows)
- Counts are reproducible directly from the DR1 tables without manual adjustment

## Limitations

- Validation depends on the DR1 release version; the published inventory (19,633/362; 18,642/345) covers the 2016-2020 GTO period only
- Category assignment relies on correct interpretation of the status/provenance columns in the DR1 planet tables; ambiguous provenance fields can split or merge categories incorrectly

## Evidence

- [abstract] DR1 makes public all GTO observations: "We present the CARMENES Data Release 1 (DR1), which makes public all observations obtained during the GTO of the CARMENES survey."
- [abstract] Inventory counts 19,633 spectra / 362 targets: "spanning from 2016 to 2020, during which 19 633 spectra for a sample of 362 targets were collected"
- [abstract] High-level RV products 18,642 / 345: "The data were pipeline-processed, and high-level data products, including 18 642 precise RVs for 345 targets, were derived."
- [abstract] Planet yield breakdown to tallied: "including 33 new planets, 17 re-analysed planets, and 26 confirmed planets from transiting candidate follow-up"
- [readme] Python-based processing context: [evidence span withheld — anchor status: external_doc]
