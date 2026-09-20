---
name: metallicity-planet-formation-correlation-analysis
description: Use when you need occurrence rates of planet classes (giants >300 M⊕,
  Earth-like, low-mass) as a function of stellar metallicity across galactic regions,
  or thin/thick disc occurrence-rate ratios in a simulated galaxy.
when_to_use_negative:
- You need absolute observational occurrence rates calibrated to real survey data
  — the low-mass planet occurrence rates are strongly model-dependent on NGPPS and
  only massive-planet findings are considered robust
- Your input is an observed stellar catalogue without simulation-based metallicity,
  age, and galactic-component labels; this skill requires the HESTIA halo/decomposition
  outputs
- You study planet formation itself rather than metallicity-driven demographic differences
  — NGPPS is applied as a fixed population model, not re-run
edam_operation: http://edamontology.org/operation_2945
tools:
- name: HESTIA galaxy formation simulation
  role: Provides the simulated Milky Way analogue and its stellar populations (metallicity,
    age, component assignment) to which the population model is applied
  repo: github.com/ChrisBoettner/skaro
- name: NGPPS (New Generation Planet Population Synthesis)
  role: Bern planet-formation models used to map stellar metallicity and mass onto
    occurrence rates of giants (>300 M⊕), Earth-like, and low-mass planets
  repo: github.com/ChrisBoettner/skaro
- name: yt
  role: Efficient processing and analysis of the HESTIA simulation snapshot when building
    the halo object and stellar/planet fields
  repo: github.com/ChrisBoettner/skaro
- name: pandas
  role: Data manipulation for aggregating occurrence rates by galactic component and
    host-star type
  repo: github.com/ChrisBoettner/skaro
- name: sklearn
  role: Machine-learning tasks supporting the planet assignment / occurrence-rate
    model
  repo: github.com/ChrisBoettner/skaro
- name: aws s3 sync / Makefile
  role: 'Data retrieval: `make sync_data_from_s3` recursively pulls raw data from
    the configured S3 bucket into data/'
  repo: github.com/ChrisBoettner/skaro
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202449557
    title: Populating the Milky Way
  source_content_hash: sha256:388f713466fe7a30c9fca7247f46247cc8d44a72f9d3447d8062667fe98040b7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HESTIA galaxy formation simulation
  - Makefile
  - NGPPS (New Generation Planet Population Synthesis)
  - aws s3 sync
  - aws s3 sync / Makefile
  - pandas
  - sklearn
  - yt
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# metallicity-planet-formation-correlation-analysis

## Summary

Quantifies how stellar metallicity ([Fe/H]) and age control planet occurrence rates by combining HESTIA simulated Milky-Way stellar populations with NGPPS (Bern) planet population synthesis, stratified by galactic component (bulge, thin disc, thick disc, stellar halo) and host-star mass (Sun-like vs 0.3–0.5 M⊙ dwarfs).

## When to use

Use when you need occurrence rates of planet classes (giants >300 M⊕, Earth-like, low-mass) as a function of stellar metallicity across galactic regions, or thin/thick disc occurrence-rate ratios in a simulated galaxy.

## When NOT to use

- You need absolute observational occurrence rates calibrated to real survey data — the low-mass planet occurrence rates are strongly model-dependent on NGPPS and only massive-planet findings are considered robust
- Your input is an observed stellar catalogue without simulation-based metallicity, age, and galactic-component labels; this skill requires the HESTIA halo/decomposition outputs
- You study planet formation itself rather than metallicity-driven demographic differences — NGPPS is applied as a fixed population model, not re-run

## Inputs

- HESTIA simulation snapshot of a Milky Way analogue with per-star metallicity, age, and Galactic component assignment (data/raw)
- NGPPS (Bern) planet population synthesis model outputs mapping stellar mass and metallicity to planet occurrence (data/external)
- Stellar parameter data and HZ coefficients (data/external)

## Outputs

- Occurrence rates of giant (>300 M⊕), Earth-like, and low-mass planets aggregated by bulge, thin disc, thick disc, and stellar halo
- Thin-disc/thick-disc occurrence-rate ratios, stratified by host-star mass regime (Sun-like vs 0.3–0.5 M⊙ dwarfs)
- Robustness classification (massive-planet ratios robust; low-mass-planet rates model-dependent)
- Planet distribution maps, radial profiles of planet density, and comparison tables (e.g. occurrence_rate_comparison.csv)

## How to apply

Retrieve the raw data with `make sync_data_from_s3` (aws s3 sync into data/), then load HESTIA stellar populations with each star's metallicity, age, and Galactic component assignment. Apply the NGPPS Bern planet population synthesis models as a planet-assignment step connecting host-star mass and metallicity to occurrence rates for giants (>300 M⊕), Earth-like planets around Sun-like stars, and low-mass planets. Aggregate predicted occurrence rates per galactic component (bulge, thin disc, thick disc, stellar halo) using the Model class in src/skaro/model.py, which loads the simulation snapshot, adds stellar and planet fields, and performs galaxy decomposition. Compare metal-rich regions (bulge, thin disc) with metal-poor regions (thick disc, stellar halo) by computing thin/thick-disc occurrence-rate ratios, and repeat stratified by host type to test whether regional contrast is stronger around Sun-like stars than 0.3–0.5 M⊙ dwarfs.

## Related tools

- **HESTIA galaxy formation simulation** (Provides the simulated Milky Way analogue and its stellar populations (metallicity, age, component assignment) to which the population model is applied) — github.com/ChrisBoettner/skaro
- **NGPPS (New Generation Planet Population Synthesis)** (Bern planet-formation models used to map stellar metallicity and mass onto occurrence rates of giants (>300 M⊕), Earth-like, and low-mass planets) — github.com/ChrisBoettner/skaro
- **yt** (Efficient processing and analysis of the HESTIA simulation snapshot when building the halo object and stellar/planet fields) — github.com/ChrisBoettner/skaro
- **pandas** (Data manipulation for aggregating occurrence rates by galactic component and host-star type) — github.com/ChrisBoettner/skaro
- **sklearn** (Machine-learning tasks supporting the planet assignment / occurrence-rate model) — github.com/ChrisBoettner/skaro
- **aws s3 sync / Makefile** (Data retrieval: `make sync_data_from_s3` recursively pulls raw data from the configured S3 bucket into data/) — github.com/ChrisBoettner/skaro

## Examples

```
git clone https://github.com/ChrisBoettner/skaro.git && mamba env create -f environment.yaml && make sync_data_from_s3
```

## Evaluation signals

- Giant-planet (>300 M⊕) occurrence rate is 10–20 times larger in the thin disc than in the thick disc
- Low-mass Earth-like planets around Sun-like stars are ~1.5 times more frequent in the thick disc than in the thin disc
- Regional (thin/thick disc) contrast is larger for Sun-like stars than for 0.3–0.5 M⊙ dwarfs, reflecting the weaker [Fe/H]–planet-mass correlation for the latter
- Low-mass planet occurrence rates are flagged as strongly model-dependent while massive-planet entries are flagged as robust
- Galaxy decomposition yields the four expected components (bulge, thin disc, thick disc, stellar halo) before aggregation

## Limitations

- Occurrence rates of low-mass planets are still uncertain, making low-mass findings strongly model-dependent; massive-planet findings are more robust
- Low-mass planets are expected to be abundant throughout the galaxy (central regions to outer halo) because their formation is less dependent on stellar metallicity, so regional contrasts for them are weak
- Raw HESTIA snapshots must be placed in data/raw and external NGPPS/stellar-parameter/HZ-coefficient files in data/external; contact the authors for data access
- Setup depends on S3 sync configuration (optional bucket) and a conda/mamba environment defined in environment.yaml

## Evidence

- [abstract] The occurrence rate of giant planets (>300 M_Earth) is 10–20 times larger in the thin disc compared to the thick disc: "The occurrence rate of giant planets (>300 M ⊕ ) is 10–20 times larger in the thin disc compared to the thick disc"
- [abstract] Earth-like planets more frequent in thick disc: "low-mass Earth-like planets around Sun-like stars are most abundant in the thick disc, being 1.5 times more frequent than in the thin disc"
- [abstract] Populations differ more around Sun-like stars: "planet populations differ more strongly around Sun-like stars compared to dwarfs with masses 0.3–0.5 M ⊙"
- [intro] Combination of HESTIA and NGPPS: "This project combines the HESTIA suite of high-resolution constrained hydrodynamical galaxy formation simulations with the New Generation Planet Population Synthesis (NGPPS) models from Bern"
- [methods] Data sync command: "`make sync_data_from_s3` will use `aws s3 sync` to recursively sync files from `s3://[OPTIONAL] your-bucket-for-syncing-data (do not include 's3://')/data/` to `data/`."
- [readme] Model class and data handling stack: [evidence span withheld — anchor status: external_doc]
