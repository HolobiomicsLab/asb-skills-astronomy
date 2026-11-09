---
name: planet-occurrence-rate-aggregation-per-region
description: Use when you have planet population synthesis grids tied to stellar mass/metallicity
  and a simulated or observed stellar population with metallicity, age, and position,
  and need occurrence rates per Galactic region and planet type.
when_to_use_negative:
- Input is an observed exoplanet transit/radial-velocity catalogue rather than population-synthesis
  grids matched to a stellar population — occurrence rates here come from NGPPS model
  matching, not detection statistics
- You need occurrence rates for a single stellar population without a Galactic-component
  decomposition (bulge/thin/thick disc/halo) — the aggregation is defined per region
- The stellar population lacks metallicity and age information, since the planet assignment
  model connects host-star mass and metallicity to planet occurrence rates
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: NGPPS (New Generation Planet Population Synthesis)
  role: Bern planet population synthesis models whose grids supply occurrence probabilities
    matched to each star's metallicity, age, and mass
- name: HESTIA galaxy formation simulation
  role: Constrained hydrodynamical simulations providing the simulated Milky Way analogue
    stellar population being populated with planets
- name: yt
  role: Efficient processing and analysis of the HESTIA simulation snapshot data
  repo: https://github.com/ChrisBoettner/skaro
- name: pandas
  role: Data manipulation for the per-star planet assignments and per-region occurrence-rate
    aggregation
  repo: https://github.com/ChrisBoettner/skaro
- name: sklearn
  role: Machine learning tasks supporting the planet assignment / galaxy decomposition
    workflow
  repo: https://github.com/ChrisBoettner/skaro
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202449557
    title: Populating the Milky Way
  source_content_hash: sha256:60adaee6759612a1ec3c6532b81dc35058d282e6711f69f0ec94e81ad953aac9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChrisBoettner/skaro
  tools:
  - HESTIA galaxy formation simulation
  - NGPPS (New Generation Planet Population Synthesis)
  - pandas
  - sklearn
  - yt
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# planet-occurrence-rate-aggregation-per-region

## Summary

Aggregates NGPPS planet population synthesis occurrence probabilities over HESTIA simulated Milky Way analogue stellar populations to compute per-region, per-planet-type occurrence rates (bulge, thin disc, thick disc, stellar halo) for planet classes such as giants >300 M⊕ and low-mass Earth-like planets.

## When to use

Use when you have planet population synthesis grids tied to stellar mass/metallicity and a simulated or observed stellar population with metallicity, age, and position, and need occurrence rates per Galactic region and planet type.

## When NOT to use

- Input is an observed exoplanet transit/radial-velocity catalogue rather than population-synthesis grids matched to a stellar population — occurrence rates here come from NGPPS model matching, not detection statistics
- You need occurrence rates for a single stellar population without a Galactic-component decomposition (bulge/thin/thick disc/halo) — the aggregation is defined per region
- The stellar population lacks metallicity and age information, since the planet assignment model connects host-star mass and metallicity to planet occurrence rates

## Inputs

- NGPPS (New Generation Planet Population Synthesis, Bern) planet population catalogs for different planet types and stellar host masses
- HESTIA simulated Milky Way analogue stellar data (metallicity, age, stellar mass, Galactocentric position per star/stellar particle)
- Stellar parameter data and habitable-zone (HZ) coefficients stored in data/external

## Outputs

- Per-region, per-planet-type occurrence-rate table (occurrence_rates.csv) for bulge, thin disc, thick disc, stellar halo
- Planet occurrence probabilities assigned per HESTIA star for each planet class (Earth-like, Super-Earths, Neptunians, Giants)
- Galaxy decomposition of stars into bulge, thin disc, thick disc, and halo components

## How to apply

Load the NGPPS planet population catalogs covering different planet types and stellar host masses, and the HESTIA simulated Milky Way analogue stellar data, extracting metallicity, age, stellar mass, and Galactocentric position per star/stellar particle. Classify each star into bulge, thin disc, thick disc, or stellar halo using age- and metallicity-based decomposition (the skaro Model class in src/skaro/model.py handles snapshot loading, halo creation, and galaxy decomposition). For every star, match its metallicity, age, and mass against the NGPPS models to assign planet occurrence probabilities for each planet class (e.g. giants >300 M⊕, low-mass Earth-like planets), then aggregate the matched probabilities over all stars in each region. Write the per-region, per-planet-type occurrence-rate table (e.g. occurrence_rates.csv) and validate against known contrasts, such as the thin-disc-to-thick-disc giant-planet ratio.

## Related tools

- **NGPPS (New Generation Planet Population Synthesis)** (Bern planet population synthesis models whose grids supply occurrence probabilities matched to each star's metallicity, age, and mass)
- **HESTIA galaxy formation simulation** (Constrained hydrodynamical simulations providing the simulated Milky Way analogue stellar population being populated with planets)
- **yt** (Efficient processing and analysis of the HESTIA simulation snapshot data) — https://github.com/ChrisBoettner/skaro
- **pandas** (Data manipulation for the per-star planet assignments and per-region occurrence-rate aggregation) — https://github.com/ChrisBoettner/skaro
- **sklearn** (Machine learning tasks supporting the planet assignment / galaxy decomposition workflow) — https://github.com/ChrisBoettner/skaro

## Examples

```
mamba env create -f environment.yaml && mamba activate skaro && pip install -e . ; from skaro.model import Model  # loads HESTIA snapshot, adds stellar/planet fields, runs galaxy decomposition
```

## Evaluation signals

- Verify the computed giant-planet (>300 M⊕) thin-disc-to-thick-disc occurrence-rate ratio is consistent with the reported 10–20x enhancement
- Verify Earth-like low-mass planets around Sun-like stars come out ~1.5x more frequent in the thick disc than the thin disc
- Check every HESTIA star is assigned to exactly one Galactic region (bulge, thin disc, thick disc, stellar halo) with no unclassified stars
- Check the output table has one row per (region, planet type) pair covering Earth-like, Super-Earths, Neptunians, and Giants, and for different host star masses
- Confirm occurrence rates vary with metallicity in the expected direction: low-mass planets abundant from central regions to outer halo due to weaker metallicity dependence

## Limitations

- Occurrence rates of low-mass planets are still uncertain, making low-mass planet findings strongly model-dependent, while massive-planet findings are more robust
- Planet populations differ more strongly around Sun-like stars than around 0.3–0.5 M⊙ dwarfs, so results are host-mass dependent
- The method requires raw HESTIA snapshots in data/raw and NGPPS/stellar-parameter/HZ-coefficient data in data/external; contact the authors for the raw data

## Evidence

- [abstract] Giant planet occurrence 10–20x thin vs thick disc: "The occurrence rate of giant planets (>300 M ⊕ ) is 10–20 times larger in the thin disc compared to the thick disc"
- [abstract] Earth-like planets 1.5x more frequent in thick disc: "low-mass Earth-like planets around Sun-like stars are most abundant in the thick disc, being 1.5 times more frequent than in the thin disc"
- [abstract] Low-mass planets abundant throughout galaxy: "low-mass planets are expected to be abundant throughout the galaxy, from the central regions to the outer halo, due to their formation processes being less dependent on stellar metallicity"
- [readme] Planet assignment model connects host properties to occurrence rates: [evidence span withheld — anchor status: external_doc]
- [readme] Model class entry point handles decomposition: [evidence span withheld — anchor status: external_doc]
- [readme] Galactic decomposition into components: [evidence span withheld — anchor status: external_doc]
