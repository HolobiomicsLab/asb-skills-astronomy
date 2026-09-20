---
name: exoplanet-occurrence-rate-estimation
description: Use when estimating planet occurrence rates as a function of stellar
  metallicity, age, or galactic population for a resolved stellar population, or when
  comparing planet demographics between metal-rich and metal-poor galactic regions.
when_to_use_negative:
- Input is an observed catalogue of detected exoplanets with detection completeness
  — this skill models occurrence from synthetic stellar populations, not from survey
  data
- The galaxy or stellar sample is not decomposable into bulge/thin disc/thick disc/halo
  components, so component-resolved rate ratios cannot be computed
edam_operation: http://edamontology.org/operation_0306
tools:
- name: HESTIA galaxy formation simulation
  role: Provides the high-resolution constrained hydrodynamical simulated Milky Way
    analogue whose stellar populations are populated with planets
  repo: github.com/ChrisBoettner/skaro
- name: NGPPS (New Generation Planet Population Synthesis)
  role: Bern planet formation model mapping host star mass and metallicity onto occurrence
    rates of different planet classes
  repo: github.com/ChrisBoettner/skaro
- name: yt
  role: Efficient processing and analysis of HESTIA simulation snapshot data
  repo: github.com/ChrisBoettner/skaro
- name: pandas
  role: Data manipulation for stellar and planet population tables
  repo: github.com/ChrisBoettner/skaro
- name: aws s3 sync
  role: Recursive retrieval/sync of raw project data via Makefile targets sync_data_from_s3
    / sync_data_to_s3
  repo: github.com/ChrisBoettner/skaro
- name: Makefile
  role: Central entry points for common project tasks including data sync, formatting,
    linting, and type checking
  repo: github.com/ChrisBoettner/skaro
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202449557
    title: Populating the Milky Way
  source_content_hash: sha256:92b706003978a5b90db0fdbb05949e7428211306c177b925a15769197c40c25f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HESTIA galaxy formation simulation
  - Makefile
  - NGPPS (New Generation Planet Population Synthesis)
  - aws s3 sync
  - pandas
  - yt
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# exoplanet-occurrence-rate-estimation

## Summary

Estimates exoplanet occurrence rates across galactic components (bulge, thin disc, thick disc, stellar halo) by combining HESTIA Milky Way analogue simulations with the Bern NGPPS planet population synthesis, mapping stellar metallicity and age onto planet type frequencies.

## When to use

Use when estimating planet occurrence rates as a function of stellar metallicity, age, or galactic population for a resolved stellar population, or when comparing planet demographics between metal-rich and metal-poor galactic regions.

## When NOT to use

- Input is an observed catalogue of detected exoplanets with detection completeness — this skill models occurrence from synthetic stellar populations, not from survey data
- The galaxy or stellar sample is not decomposable into bulge/thin disc/thick disc/halo components, so component-resolved rate ratios cannot be computed

## Inputs

- HESTIA simulation snapshot (stellar metallicity, age, positions, galactic component assignment) in data/raw
- NGPPS planet population synthesis tables in data/external
- Stellar parameter data and HZ coefficients in data/external
- Host star masses (e.g. Sun-like vs 0.3-0.5 M_solar dwarfs) and metallicities

## Outputs

- Occurrence rates of giant (>300 M_Earth), Earth-like, Super-Earth, Neptunian, and low-mass planets per galactic component (bulge, thin disc, thick disc, halo)
- Thin/thick disc occurrence-rate ratios
- Maps of planet distributions and radial profiles of planet density in figures/

## How to apply

Retrieve raw simulation data with `make sync_data_from_s3` (aws s3 sync into data/), then load the HESTIA snapshot via the `Model` class in src/skaro/model.py, which adds stellar and planet fields and performs the galactic decomposition into bulge, thin disc, thick disc, and halo. Use the Planet Assignment Model to connect host star mass and metallicity to NGPPS-predicted occurrence rates for giants (>300 M_Earth), Earth-like planets around Sun-like stars, and low-mass planets. Aggregate occurrence rates per galactic component and compute cross-component ratios (e.g. thin/thick disc), exploiting the fact that low-mass planet formation depends weakly on metallicity while giant planet formation requires metal-rich stars. Validate against the paper's benchmark ratios and handle yt/pandas data structures; the pipeline is orchestrated through the Makefile.

## Related tools

- **HESTIA galaxy formation simulation** (Provides the high-resolution constrained hydrodynamical simulated Milky Way analogue whose stellar populations are populated with planets) — github.com/ChrisBoettner/skaro
- **NGPPS (New Generation Planet Population Synthesis)** (Bern planet formation model mapping host star mass and metallicity onto occurrence rates of different planet classes) — github.com/ChrisBoettner/skaro
- **yt** (Efficient processing and analysis of HESTIA simulation snapshot data) — github.com/ChrisBoettner/skaro
- **pandas** (Data manipulation for stellar and planet population tables) — github.com/ChrisBoettner/skaro
- **aws s3 sync** (Recursive retrieval/sync of raw project data via Makefile targets sync_data_from_s3 / sync_data_to_s3) — github.com/ChrisBoettner/skaro
- **Makefile** (Central entry points for common project tasks including data sync, formatting, linting, and type checking) — github.com/ChrisBoettner/skaro

## Examples

```
make sync_data_from_s3
```

## Evaluation signals

- Giant planet (>300 M_Earth) occurrence rate comes out 10-20 times larger in the thin disc than in the thick disc
- Low-mass Earth-like planets around Sun-like stars are ~1.5 times more frequent in the thick disc than in the thin disc
- Galaxy decomposition cleanly separates stars into bulge, thin disc, thick disc, and stellar halo components
- Data retrieval completes with the expected HESTIA snapshots in data/raw and NGPPS tables in data/external
- Occurrence rates are computed separately per host star mass regime (Sun-like vs 0.3-0.5 M_solar dwarfs)

## Limitations

- Occurrence rates of low-mass planets are still uncertain, making findings for Earth-like and low-mass planets strongly model-dependent
- Massive (giant) planet findings are more robust because their formation is better understood
- Raw HESTIA snapshots are not in the repository — users must sync from S3 or contact the author for data access

## Evidence

- [abstract] Giant occurrence benchmark: "The occurrence rate of giant planets (>300 M ⊕ ) is 10–20 times larger in the thin disc compared to the thick disc"
- [abstract] Earth-like benchmark: "low-mass Earth-like planets around Sun-like stars are most abundant in the thick disc, being 1.5 times more frequent than in the thin disc"
- [methods] Data sync step: "`make sync_data_from_s3` will use `aws s3 sync` to recursively sync files from `s3://[OPTIONAL] your-bucket-for-syncing-data (do not include 's3://')/data/` to `data/`."
- [readme] Model entry point: [evidence span withheld — anchor status: external_doc]
- [abstract] Low-mass metallicity dependence: "low-mass planets are expected to be abundant throughout the galaxy, from the central regions to the outer halo, due to their formation processes being less dependent on stellar metallicity"
