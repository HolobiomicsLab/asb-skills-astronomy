---
name: galactic-disc-component-classification-thick-thin-halo
description: Use when you have a simulated stellar population with per-star metallicity,
  age, stellar mass, and Galactocentric position, and need to decompose it into bulge,
  thin disc, thick disc, and halo components before aggregating demographics per region.
when_to_use_negative:
- Input is an observed stellar catalogue without the simulated quantities (age, metallicity,
  Galactocentric position) that the age/metallicity classification requires
- The analysis goal is a single population-integrated planet occurrence rate with
  no Galactic decomposition — this step adds decomposition you do not need
- The target galaxy is not a Milky Way analogue with distinguishable thin/thick disc
  components (e.g. dwarf or elliptical galaxies)
tools:
- name: yt
  role: Efficient loading and processing of HESTIA simulation snapshots to expose
    stellar fields used for decomposition
  repo: github.com/ChrisBoettner/skaro
- name: pandas
  role: Data manipulation of the per-star decomposition and occurrence-rate tables
  repo: github.com/ChrisBoettner/skaro
- name: sklearn
  role: Machine-learning tasks supporting the galaxy/stellar-component classification
  repo: github.com/ChrisBoettner/skaro
- name: HESTIA galaxy formation simulation
  role: Source of the simulated Milky Way analogue stellar populations being decomposed
- name: Model class (src/skaro/model.py)
  role: Main entry point that loads the snapshot, creates the halo object, adds stellar
    and planet fields, and calculates the galaxy decomposition
  repo: github.com/ChrisBoettner/skaro
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202449557
    title: Populating the Milky Way
  source_content_hash: sha256:941034cc39b4fe346b0217076c5b7b63a432603caa92bee5e14d64c2d143eec1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HESTIA galaxy formation simulation
  - Model class (src/skaro/model.py)
  - pandas
  - sklearn
  - yt
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# galactic-disc-component-classification-thick-thin-halo

## Summary

Assigns each star or stellar particle in a simulated Milky Way analogue (e.g. from HESTIA) to a Galactic component — bulge, thin disc, thick disc, or stellar halo — using age- and metallicity-based classification, so that planet occurrence rates can be computed per stellar environment.

## When to use

Use when you have a simulated stellar population with per-star metallicity, age, stellar mass, and Galactocentric position, and need to decompose it into bulge, thin disc, thick disc, and halo components before aggregating demographics per region.

## When NOT to use

- Input is an observed stellar catalogue without the simulated quantities (age, metallicity, Galactocentric position) that the age/metallicity classification requires
- The analysis goal is a single population-integrated planet occurrence rate with no Galactic decomposition — this step adds decomposition you do not need
- The target galaxy is not a Milky Way analogue with distinguishable thin/thick disc components (e.g. dwarf or elliptical galaxies)

## Inputs

- HESTIA simulated Milky Way analogue snapshot (raw simulation data in data/raw/)
- Per-star/stellar-particle fields: metallicity, age, stellar mass, Galactocentric position
- NGPPS-derived planet occurrence probabilities keyed to stellar metallicity, age, and mass (data/external)

## Outputs

- Per-star Galactic-component assignment (bulge / thin disc / thick disc / stellar halo)
- Halo information and galaxy decomposition results saved to data/processed
- Per-region star sets enabling per-region, per-planet-type occurrence-rate tables (e.g. occurrence_rates.csv)

## How to apply

Load the HESTIA Milky Way analogue snapshot (raw snapshots in data/raw) using the Model class in src/skaro/model.py, which creates the halo object and adds stellar and planet fields before calculating the galaxy decomposition. Extract metallicity, age, stellar mass, and Galactocentric position for each star/stellar particle, then classify each star into one of four regions — bulge, thin disc, thick disc, stellar halo — using age- and metallicity-based criteria, since the thick disc and stellar halo are the older, more metal-poor components versus the metal-rich bulge and thin disc. The repo leverages yt for snapshot processing, pandas for manipulating the decomposition tables, and sklearn for machine-learning tasks in the classification step. Downstream, the per-region star sets become the denominator for NGPPS planet-occurrence aggregation (e.g. giants >300 M⊕, Earth-like low-mass planets), so a per-star unique component assignment is the key intermediate output.

## Related tools

- **yt** (Efficient loading and processing of HESTIA simulation snapshots to expose stellar fields used for decomposition) — github.com/ChrisBoettner/skaro
- **pandas** (Data manipulation of the per-star decomposition and occurrence-rate tables) — github.com/ChrisBoettner/skaro
- **sklearn** (Machine-learning tasks supporting the galaxy/stellar-component classification) — github.com/ChrisBoettner/skaro
- **HESTIA galaxy formation simulation** (Source of the simulated Milky Way analogue stellar populations being decomposed)
- **Model class (src/skaro/model.py)** (Main entry point that loads the snapshot, creates the halo object, adds stellar and planet fields, and calculates the galaxy decomposition) — github.com/ChrisBoettner/skaro

## Examples

```
python -c "from skaro.model import Model; m = Model(); m.load_snapshot('data/raw'); m.galaxy_decomposition()"
```

## Evaluation signals

- Every HESTIA star/stellar particle receives exactly one component label from the set {bulge, thin disc, thick disc, stellar halo}
- Component assignments are consistent with age/metallicity structure: thick disc and stellar halo stars are older and more metal-poor than bulge/thin disc stars
- Downstream validation: giant planet (>300 M⊕) thin-disc-to-thick-disc occurrence-rate ratio falls in the reported 10–20x range
- Downstream validation: Earth-like low-mass planets around Sun-like stars come out ~1.5x more frequent in the thick disc than the thin disc
- Decomposition results are reproducibly written to data/processed as expected by downstream aggregation code

## Limitations

- Occurrence rates of low-mass planets are uncertain, making downstream findings strongly model-dependent; massive planet results are more robust
- The classification is calibrated for a Milky Way analogue from the HESTIA suite; transferring to other galaxy simulations may require re-tuning age/metallicity boundaries
- Raw HESTIA snapshots are assumed in data/raw and external NGPPS/stellar-parameter/HZ-coefficient inputs in data/external — contact the author for data access

## Evidence

- [methods] Assign each HESTIA star to a Galactic region (bulge, thin disc, thick disc, stellar halo) using age and metallicity-based classification.: [evidence span withheld — anchor status: not_found]
- [methods] Load the HESTIA simulated Milky Way analogue stellar data, extracting metallicity, age, stellar mass, and Galactocentric position for each star/stellar particle.: [evidence span withheld — anchor status: not_found]
- [readme] Divides the simulated galaxy into its major components (bulge, thin disk, thick disk, and halo) to analyze planet demographics in distinct stellar environments.: [evidence span withheld — anchor status: external_doc]
- [readme] The main entry point for running the code is the Model class in src/skaro/model.py. It handles loading the simulation snapshot, creating the halo object.: [evidence span withheld — anchor status: external_doc]
