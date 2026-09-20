---
name: stellar-age-metallicity-distribution-mapping
description: Use when a simulated stellar population (e.g. a HESTIA snapshot) must
  be divided into galactic components (bulge, thin disk, thick disk, halo) via age-
  and metallicity-based classification before assigning planet occurrence rates per
  region.
when_to_use_negative:
- You already have a per-region planet occurrence-rate table and only need planet-population
  interpretation — this skill operates upstream on stellar populations.
- The stellar catalog lacks per-star age and metallicity, so the age/metallicity-based
  galactic decomposition cannot be performed.
- You need to generate the planet population synthesis itself (NGPPS-style Bern model
  runs) rather than map stars onto an existing synthesis.
tools:
- name: HESTIA galaxy formation simulation
  role: Source of the simulated Milky Way analogue snapshots whose stellar populations
    are decomposed by age and metallicity
- name: yt
  role: Efficient reading and processing of HESTIA simulation snapshot data
- name: pandas
  role: Manipulation of per-star stellar parameter tables (metallicity, age, mass,
    position) and decomposition results
- name: sklearn
  role: Machine-learning support for the classification/decomposition tasks
- name: skaro Model class (src/skaro/model.py)
  role: Entry point that loads the snapshot, creates the halo object, adds stellar
    and planet fields, and calculates the galaxy decomposition
  repo: https://github.com/ChrisBoettner/skaro
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202449557
    title: Populating the Milky Way
  source_content_hash: sha256:d8f4754f18fb59f1a8197e481a344c9acd027aba1f6b946e5ad4197ec62daf2f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChrisBoettner/skaro
  tools:
  - HESTIA galaxy formation simulation
  - pandas
  - skaro Model class (src/skaro/model.py)
  - sklearn
  - yt
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# stellar-age-metallicity-distribution-mapping

## Summary

Decomposes a simulated Milky Way analogue (HESTIA) stellar population into bulge, thin disc, thick disc, and stellar halo components using per-star age, metallicity, stellar mass, and Galactocentric position, producing the per-region stellar inventories needed for environment-dependent planet occurrence-rate mapping.

## When to use

Use when a simulated stellar population (e.g. a HESTIA snapshot) must be divided into galactic components (bulge, thin disk, thick disk, halo) via age- and metallicity-based classification before assigning planet occurrence rates per region.

## When NOT to use

- You already have a per-region planet occurrence-rate table and only need planet-population interpretation — this skill operates upstream on stellar populations.
- The stellar catalog lacks per-star age and metallicity, so the age/metallicity-based galactic decomposition cannot be performed.
- You need to generate the planet population synthesis itself (NGPPS-style Bern model runs) rather than map stars onto an existing synthesis.

## Inputs

- HESTIA hydrodynamical simulation snapshot (raw) located in data/raw
- NGPPS (Bern) planet population synthesis catalogs, stellar parameter data, and HZ coefficients stored in data/external

## Outputs

- Per-star galactic component labels (bulge, thin disk, thick disk, halo) plus halo information stored in data/processed
- Region-level stellar inventories (metallicity, age, mass, position distributions) that feed per-region, per-planet-type occurrence-rate tables such as occurrence_rates.csv
- Maps of planet distributions and radial profiles of planet density (figures directory)

## How to apply

Instantiate the `Model` class in `src/skaro/model.py` of the skaro repository, which loads the HESTIA simulation snapshot from `data/raw`, creates the halo object, and adds stellar and planet fields. Extract metallicity, age, stellar mass, and Galactocentric position for each star/stellar particle, then perform the galaxy decomposition into the four components (bulge, thin disk, thick disk, halo) using age and metallicity as the discriminating quantities. Save halo information and decomposition results to `data/processed` (NGPPS catalogs, stellar parameter data, and HZ coefficients live in `data/external`). Use `yt` to read the snapshot data, `pandas` for the per-star tables, and `sklearn` for classification/ML tasks; the Makefile provides convenience entry points for setup and data syncing (`make sync_data_to_s3` / `make sync_data_from_s3`). The resulting per-star component labels are the grouping key for aggregating NGPPS-matched planet occurrence probabilities into per-region rates.

## Related tools

- **HESTIA galaxy formation simulation** (Source of the simulated Milky Way analogue snapshots whose stellar populations are decomposed by age and metallicity)
- **yt** (Efficient reading and processing of HESTIA simulation snapshot data)
- **pandas** (Manipulation of per-star stellar parameter tables (metallicity, age, mass, position) and decomposition results)
- **sklearn** (Machine-learning support for the classification/decomposition tasks)
- **skaro Model class (src/skaro/model.py)** (Entry point that loads the snapshot, creates the halo object, adds stellar and planet fields, and calculates the galaxy decomposition) — https://github.com/ChrisBoettner/skaro

## Examples

```
mamba env create -f environment.yaml && mamba activate skaro && pip install -e .
```

## Evaluation signals

- Every star/stellar particle in the snapshot receives exactly one component label among bulge, thin disk, thick disk, and halo, and component stellar masses sum to the total stellar mass.
- Downstream giant-planet (>300 M⊕) occurrence rates reproduce the reported 10–20x thin-disc-to-thick-disc enhancement.
- Earth-like low-mass planets around Sun-like stars come out ~1.5x more frequent in the thick disc than the thin disc.
- Radial profiles of planet density and distribution maps are internally consistent with the assigned galactic geometry (bulge innermost, halo outermost).
- Decomposition results and halo information are written to data/processed and reproducible via the skaro environment (environment.yaml).

## Limitations

- Raw HESTIA snapshots are not distributed with the repository — they are assumed to be in data/raw, and the README says 'Please contact me for more information', so data access is a prerequisite.
- The decomposition is based on the simulated galaxy's own age/metallicity structure; classification of borderline stars into thin vs. thick disc components introduces assignment uncertainty.
- Occurrences of low-mass planets built on these regions are strongly model-dependent (uncertain), whereas massive-planet results are more robust.

## Evidence

- [readme] Galactic Decomposition: Divides the simulated galaxy into its major components (bulge, thin disk, thick disk, and halo) to analyze planet demographics in distinct stellar environments.: [evidence span withheld — anchor status: not_found]
- [readme] Model class entry point: [evidence span withheld — anchor status: external_doc]
- [readme] yt/pandas/sklearn stack: [evidence span withheld — anchor status: external_doc]
- [abstract] Abstract combination statement: "based on the New Generation Planet Population Synthesis (NGPPS)"
- [intro] Intro project description: "This project combines the HESTIA suite of high-resolution constrained hydrodynamical galaxy formation simulations"
- [abstract] Abstract key result used as validation: "The occurrence rate of giant planets (>300 M ⊕ ) is 10–20 times larger in the thin disc compared to the thick disc"
