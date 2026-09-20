---
name: planet-host-star-matching-across-simulation-and-synthesis-catalogs
description: Use when you have stellar catalogs (metallicity, age, mass) from a galaxy
  simulation and need to estimate per-star or per-region planet occurrence rates from
  a planet population synthesis model such as NGPPS.
when_to_use_negative:
- You need absolute planet counts rather than occurrence-rate estimates tied to a
  specific synthesis model — the low-mass planet results are strongly model-dependent
- Your stellar catalog lacks metallicity and/or stellar mass, since the planet assignment
  model matches hosts on mass and metallicity
- You are studying planet occurrence in observed stellar samples rather than simulation
  particles — HESTIA-specific positional data is used for the Galactic decomposition
tools:
- name: NGPPS (New Generation Planet Population Synthesis)
  role: Bern planet population synthesis models providing occurrence-rate grids matched
    against host-star properties
- name: HESTIA
  role: Constrained hydrodynamical galaxy formation simulations supplying the Milky
    Way analogue stellar populations being populated with planets
- name: yt
  role: Efficient processing and analysis of HESTIA simulation snapshot data
  repo: https://github.com/ChrisBoettner/skaro
- name: pandas
  role: Data manipulation of stellar and planet catalogs during matching and aggregation
  repo: https://github.com/ChrisBoettner/skaro
- name: sklearn
  role: Machine learning tasks supporting the stellar/planet classification pipeline
  repo: https://github.com/ChrisBoettner/skaro
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202449557
    title: Populating the Milky Way
  source_content_hash: sha256:2e502aa2940bdc2f391fcf4dbad35a13822fd1c22c32638272d5cc75b5d3ee4c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChrisBoettner/skaro
  tools:
  - HESTIA
  - HESTIA galaxy formation simulation
  - NGPPS (New Generation Planet Population Synthesis)
  - pandas
  - sklearn
  - yt
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# planet-host-star-matching-across-simulation-and-synthesis-catalogs

## Summary

Assigns planet occurrence probabilities to simulated stars by matching each HESTIA star's metallicity, age, and mass against NGPPS (Bern) planet population synthesis models, then aggregating per planet class across Galactic regions. This bridges galaxy-formation simulation output with planet population statistics.

## When to use

Use when you have stellar catalogs (metallicity, age, mass) from a galaxy simulation and need to estimate per-star or per-region planet occurrence rates from a planet population synthesis model such as NGPPS.

## When NOT to use

- You need absolute planet counts rather than occurrence-rate estimates tied to a specific synthesis model — the low-mass planet results are strongly model-dependent
- Your stellar catalog lacks metallicity and/or stellar mass, since the planet assignment model matches hosts on mass and metallicity
- You are studying planet occurrence in observed stellar samples rather than simulation particles — HESTIA-specific positional data is used for the Galactic decomposition

## Inputs

- HESTIA simulation snapshot (raw data in `data/raw`) containing stellar particles with metallicity, age, stellar mass, and Galactocentric position
- NGPPS (New Generation Planet Population Synthesis, Bern) planet population catalogs covering different planet types and stellar host masses (stored in `data/external`)
- Stellar parameter data and HZ coefficients (`data/external`)

## Outputs

- Per-star planet occurrence probabilities for each planet class (Earth-like, Super-Earth, Neptunian, Giant >300 M⊕)
- Galaxy decomposition table assigning stars to bulge, thin disk, thick disk, and stellar halo (saved in `data/processed`)
- Per-region, per-planet-type occurrence-rate table (occurrence_rates.csv)

## How to apply

Load the HESTIA Milky Way analogue snapshot via the `Model` class in `src/skaro/model.py`, which loads the snapshot, creates the halo object, and adds stellar and planet fields. Classify each star into a Galactic component (bulge, thin disk, thick disk, halo) using age- and metallicity-based decomposition. For every star, match its host-star mass and metallicity against the NGPPS population grids to assign occurrence probabilities per planet class (giants >300 M⊕, Neptunians, Super-Earths, low-mass Earth-like planets), leveraging the Planet Assignment Model that 'connects the properties of host stars (mass and metallicity) with the occurrence rates of different planet types'. Aggregate the matched probabilities over all stars in each Galactic component to compute per-region, per-planet-type occurrence rates, exploiting `yt` for snapshot processing and `pandas` for catalog manipulation.

## Related tools

- **NGPPS (New Generation Planet Population Synthesis)** (Bern planet population synthesis models providing occurrence-rate grids matched against host-star properties)
- **HESTIA** (Constrained hydrodynamical galaxy formation simulations supplying the Milky Way analogue stellar populations being populated with planets)
- **yt** (Efficient processing and analysis of HESTIA simulation snapshot data) — https://github.com/ChrisBoettner/skaro
- **pandas** (Data manipulation of stellar and planet catalogs during matching and aggregation) — https://github.com/ChrisBoettner/skaro
- **sklearn** (Machine learning tasks supporting the stellar/planet classification pipeline) — https://github.com/ChrisBoettner/skaro

## Examples

```
from skaro.model import Model; m = Model(snapshot); m.add_stars(); m.add_planets(); m.decompose()
```

## Evaluation signals

- Reproduce the reported validation anchor: giant-planet (>300 M⊕) thin-disc-to-thick-disc occurrence-rate ratio must fall in the 10–20x range
- Check that low-mass Earth-like planets around Sun-like stars come out ~1.5x more frequent in the thick disc than the thin disc
- Verify the Galactic decomposition assigns every star to exactly one of bulge, thin disk, thick disk, or halo using age and metallicity criteria
- Confirm occurrence rates are aggregated only from stars whose mass and metallicity lie within the NGPPS model coverage; unmatched stars should be excluded or flagged
- Compare results separately for Sun-like stars and 0.3–0.5 M⊙ dwarfs — differences should be stronger around Sun-like stars, per the paper's finding

## Limitations

- Occurrence rates of low-mass planets are still uncertain, making low-mass planet findings strongly model-dependent; massive-planet findings are more robust
- Planet populations differ more strongly around Sun-like stars than around 0.3–0.5 M⊙ dwarfs, so occurrence rates derived for one stellar-mass regime do not transfer to the other
- Raw HESTIA snapshots are not distributed in the repository — data must be obtained separately ('Please contact me for more information'), and paths are assumed under `data/raw`

## Evidence

- [abstract] Giant planet occurrence in thin vs thick disc: "The occurrence rate of giant planets (>300 M ⊕ ) is 10–20 times larger in the thin disc compared to the thick disc"
- [abstract] Earth-like planets in thick vs thin disc: "low-mass Earth-like planets around Sun-like stars are most abundant in the thick disc, being 1.5 times more frequent than in the thin disc"
- [intro] Project scope: "This project combines the HESTIA suite of high-resolution constrained hydrodynamical galaxy formation simulations with the New Generation Planet Population Synthesis (NGPPS) models from Bern"
- [readme] Planet assignment model: [evidence span withheld — anchor status: external_doc]
- [readme] Galactic decomposition scope: [evidence span withheld — anchor status: external_doc]
