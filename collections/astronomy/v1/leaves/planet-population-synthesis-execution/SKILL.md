---
name: planet-population-synthesis-execution
description: Use when estimating planet occurrence rates (giants >300 M⊕, Earth-like
  planets around Sun-like stars, low-mass planets) across Galactic regions (bulge,
  thin disc, thick disc, stellar halo) by mapping simulated stellar metallicity/age
  onto planet population synthesis models.
when_to_use_negative:
- When analyzing observed exoplanet transit/RV detections rather than simulated stellar
  populations — this skill predicts occurrence from synthesis models, not from survey
  statistics
- When you need robust quantitative rates for low-mass planets — the paper states
  these occurrence rates are still uncertain, making low-mass findings strongly model-dependent
  (massive-planet results are more robust)
- When the target stellar population is restricted to 0.3–0.5 M⊙ dwarfs, where planet
  populations differ much less strongly across Galactic components than around Sun-like
  stars
tools:
- name: NGPPS (New Generation Planet Population Synthesis)
  role: Bern planet formation models whose population catalogs are matched against
    each star's mass and metallicity to assign planet occurrence probabilities
- name: HESTIA galaxy formation simulation
  role: Provides the simulated Milky Way analogue stellar populations (metallicity,
    age, position) that get decomposed into Galactic components and populated with
    planets
- name: yt
  role: Efficient processing and analysis of the HESTIA simulation snapshot data
  repo: github.com/ChrisBoettner/skaro
- name: pandas
  role: Data manipulation for aggregating matched planet occurrence probabilities
    into per-region rate tables
  repo: github.com/ChrisBoettner/skaro
- name: Makefile / aws s3 sync
  role: Entry point `make sync_data_from_s3` recursively pulls raw HESTIA and NGPPS
    data from the configured S3 bucket into the local data/ directory
  repo: github.com/ChrisBoettner/skaro
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202449557
    title: Populating the Milky Way
  source_content_hash: sha256:11ab0e8880c3cf33b19b5275413910da0c99ceedbe78c4be7e8be9e5b5f1bfb2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HESTIA galaxy formation simulation
  - Makefile
  - Makefile / aws s3 sync
  - NGPPS (New Generation Planet Population Synthesis)
  - aws s3 sync
  - pandas
  - yt
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# planet-population-synthesis-execution

## Summary

Executes the Bern NGPPS planet population synthesis on the HESTIA simulated Milky Way analogue, assigning planet occurrence (giants >300 M⊕, Earth-like, low-mass planets) to every star based on host mass and metallicity, then aggregating rates per Galactic component. It underpins the paper's quantification of how planet demographics vary from the metal-rich bulge/thin disc to the metal-poor thick disc and stellar halo.

## When to use

Use when estimating planet occurrence rates (giants >300 M⊕, Earth-like planets around Sun-like stars, low-mass planets) across Galactic regions (bulge, thin disc, thick disc, stellar halo) by mapping simulated stellar metallicity/age onto planet population synthesis models.

## When NOT to use

- When analyzing observed exoplanet transit/RV detections rather than simulated stellar populations — this skill predicts occurrence from synthesis models, not from survey statistics
- When you need robust quantitative rates for low-mass planets — the paper states these occurrence rates are still uncertain, making low-mass findings strongly model-dependent (massive-planet results are more robust)
- When the target stellar population is restricted to 0.3–0.5 M⊙ dwarfs, where planet populations differ much less strongly across Galactic components than around Sun-like stars

## Inputs

- HESTIA constrained hydrodynamical simulation snapshot of a Milky Way analogue (stellar particles with metallicity, age, mass, Galactocentric position), stored in data/raw
- NGPPS (New Generation Planet Population Synthesis, Bern) population catalogs covering different planet types and stellar host masses, stored in data/external
- Stellar parameter data and HZ coefficients in data/external
- environment.yaml conda environment specification

## Outputs

- Per-region, per-planet-type occurrence-rate table (e.g. occurrence_rates.csv) for bulge, thin disc, thick disc, and stellar halo
- Galaxy decomposition of the simulated MW analogue into bulge, thin disk, thick disk, and halo
- Planet-populated galaxy maps and radial profiles of planet density in the figures/ directory

## How to apply

Retrieve the project's raw data with `make sync_data_from_s3`, which recursively pulls the HESTIA simulation snapshots (into `data/raw`) and the NGPPS catalogs, stellar parameter data, and HZ coefficients (in `data/external`) from the configured S3 bucket. Load the HESTIA snapshot through the `Model` class in `src/skaro/model.py`, which creates the halo object, adds stellar and planet fields, and performs the galaxy decomposition that splits stars into bulge, thin disc, thick disc, and stellar halo using age and metallicity classification. For every star, match its mass and metallicity against the NGPPS (Bern) population models to assign occurrence probabilities for each planet class (giants >300 M⊕, Super-Earths, Neptunians, Earth-like planets). Aggregate matched probabilities over all stars in each Galactic component, comparing metal-rich regions (bulge, thin disc) with metal-poor ones (thick disc, stellar halo), and also stratify by host mass since populations differ more strongly around Sun-like stars than around 0.3–0.5 M⊙ dwarfs. Leverage `yt` for snapshot processing, `pandas` for the aggregation tables, and `sklearn` for any classification steps, writing per-region, per-planet-type rates (e.g. to occurrence_rates.csv) and planet distribution maps/radial profiles to `figures/`.

## Related tools

- **NGPPS (New Generation Planet Population Synthesis)** (Bern planet formation models whose population catalogs are matched against each star's mass and metallicity to assign planet occurrence probabilities)
- **HESTIA galaxy formation simulation** (Provides the simulated Milky Way analogue stellar populations (metallicity, age, position) that get decomposed into Galactic components and populated with planets)
- **yt** (Efficient processing and analysis of the HESTIA simulation snapshot data) — github.com/ChrisBoettner/skaro
- **pandas** (Data manipulation for aggregating matched planet occurrence probabilities into per-region rate tables) — github.com/ChrisBoettner/skaro
- **Makefile / aws s3 sync** (Entry point `make sync_data_from_s3` recursively pulls raw HESTIA and NGPPS data from the configured S3 bucket into the local data/ directory) — github.com/ChrisBoettner/skaro

## Examples

```
mamba env create -f environment.yaml && mamba activate skaro && make sync_data_from_s3
```

## Evaluation signals

- Giant-planet (>300 M⊕) occurrence rate comes out 10–20 times larger in the thin disc than in the thick disc
- Low-mass Earth-like planets around Sun-like stars are ~1.5 times more frequent in the thick disc than in the thin disc
- Galaxy decomposition cleanly assigns every star/particle to exactly one of bulge, thin disc, thick disc, or stellar halo
- Low-mass planets appear abundant throughout the galaxy, from the central regions to the outer halo, reflecting their weaker metallicity dependence
- Output table schema covers region × planet class × host-star mass bin, with metal-rich (bulge, thin disc) vs metal-poor (thick disc, halo) comparisons reproducible from the stored data

## Limitations

- Low-mass planet occurrence rates are still uncertain, so the low-mass findings are strongly model-dependent; massive-planet findings are more robust
- Planet populations differ more strongly around Sun-like stars than around 0.3–0.5 M⊙ dwarfs, so conclusions do not transfer uniformly across host masses
- Raw HESTIA snapshots must be obtained separately (contact the authors / S3 sync); the repository does not ship them

## Evidence

- [abstract] abstract: "The occurrence rate of giant planets (>300 M ⊕ ) is 10–20 times larger in the thin disc compared to the thick disc"
- [intro] intro: "This project combines the HESTIA suite of high-resolution constrained hydrodynamical galaxy formation simulations with the New Generation Planet Population Synthesis (NGPPS) models from Bern"
- [methods] methods: "`make sync_data_from_s3` will use `aws s3 sync` to recursively sync files from"
- [readme] readme: [evidence span withheld — anchor status: external_doc]
