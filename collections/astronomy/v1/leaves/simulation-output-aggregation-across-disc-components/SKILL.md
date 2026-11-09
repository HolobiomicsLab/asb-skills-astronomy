---
name: simulation-output-aggregation-across-disc-components
description: Use when you have per-star properties (metallicity, age, Galactic component
  assignment) from a galaxy-formation simulation and need component-resolved occurrence
  rates for different planet classes rather than a galaxy-wide average.
when_to_use_negative:
- You need robust occurrence rates for low-mass planets — the article notes these
  are strongly model-dependent and uncertain; only massive-planet findings are more
  robust
- Your target is a single galaxy-wide occurrence rate without component decomposition,
  or your stellar population lacks metallicity/age information needed for the NGPPS-based
  planet assignment
- The simulation output is not a Milky Way analogue with resolvable thin/thick disc
  components
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: HESTIA galaxy formation simulation
  role: Provides the simulated Milky Way analogue snapshot whose stellar populations
    are decomposed into bulge, thin disc, thick disc, and halo
  repo: github.com/ChrisBoettner/skaro
- name: NGPPS (New Generation Planet Population Synthesis)
  role: Bern planet formation models used to map host-star mass and metallicity onto
    occurrence rates of giants, Earth-like, and low-mass planets
- name: Makefile
  role: Entry point for sync_data_from_s3 (aws s3 sync) to retrieve raw HESTIA data
    into data/, and for code formatting, linting, and type checking
- name: aws s3 sync
  role: Recursively pulls the project's raw simulation data from the configured S3
    bucket into the local data/ directory before aggregation
- name: yt
  role: Efficient processing and analysis of the HESTIA simulation snapshot data
- name: pandas
  role: Data manipulation of stellar and planet fields during component aggregation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202449557
    title: Populating the Milky Way
  source_content_hash: sha256:331eed174e48517694db50420b4c81411a5a115275bf15f905813862f48354fc
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# simulation-output-aggregation-across-disc-components

## Summary

Aggregates planet occurrence rates predicted for a simulated Milky Way analogue by galaxy component (bulge, thin disc, thick disc, stellar halo) and host-star mass, so that metal-rich versus metal-poor stellar populations can be compared quantitatively.

## When to use

Use when you have per-star properties (metallicity, age, Galactic component assignment) from a galaxy-formation simulation and need component-resolved occurrence rates for different planet classes rather than a galaxy-wide average.

## When NOT to use

- You need robust occurrence rates for low-mass planets — the article notes these are strongly model-dependent and uncertain; only massive-planet findings are more robust
- Your target is a single galaxy-wide occurrence rate without component decomposition, or your stellar population lacks metallicity/age information needed for the NGPPS-based planet assignment
- The simulation output is not a Milky Way analogue with resolvable thin/thick disc components

## Inputs

- HESTIA simulation snapshot of a Milky Way analogue (per-star metallicity, age, mass, position)
- NGPPS (Bern) planet population synthesis data stored in data/external
- Stellar parameter data and HZ coefficients in data/external

## Outputs

- Component-resolved planet occurrence rates (bulge, thin disc, thick disc, stellar halo) for giant (>300 M⊕), Earth-like, and low-mass planet classes
- Thin/thick disc occurrence-rate ratios per planet class
- Halo information and galaxy decomposition results saved to data/processed

## How to apply

Load the HESTIA simulation snapshot via the Model class in src/skaro/model.py, which loads the snapshot, creates the halo object, and adds stellar and planet fields. Perform the galactic decomposition into bulge, thin disk, thick disk, and halo as implemented in the skaro code. Map stellar mass and metallicity onto planet occurrence via the NGPPS-based planet assignment model, treating giants (>300 M⊕), Earth-like planets around Sun-like stars, and low-mass planets as separate classes. Aggregate predicted occurrence per component, then compute thin/thick disc ratios: expect giants to be 10–20× more frequent in the thin disc and low-mass Earth-like planets ~1.5× more frequent in the thick disc, reflecting the weaker metallicity dependence of low-mass planet formation.

## Related tools

- **HESTIA galaxy formation simulation** (Provides the simulated Milky Way analogue snapshot whose stellar populations are decomposed into bulge, thin disc, thick disc, and halo) — github.com/ChrisBoettner/skaro
- **NGPPS (New Generation Planet Population Synthesis)** (Bern planet formation models used to map host-star mass and metallicity onto occurrence rates of giants, Earth-like, and low-mass planets)
- **Makefile** (Entry point for sync_data_from_s3 (aws s3 sync) to retrieve raw HESTIA data into data/, and for code formatting, linting, and type checking)
- **aws s3 sync** (Recursively pulls the project's raw simulation data from the configured S3 bucket into the local data/ directory before aggregation)
- **yt** (Efficient processing and analysis of the HESTIA simulation snapshot data)
- **pandas** (Data manipulation of stellar and planet fields during component aggregation)

## Examples

```
make sync_data_from_s3 && mamba activate skaro && python -c "from skaro.model import Model; m = Model(); m.load_snapshot(); m.galaxy_decomposition()"
```

## Evaluation signals

- Giant planet (>300 M⊕) occurrence rate comes out 10–20 times larger in the thin disc than in the thick disc
- Low-mass Earth-like planets around Sun-like stars are ~1.5 times more frequent in the thick disc than in the thin disc
- Galaxy decomposition cleanly separates every star into exactly one of bulge, thin disc, thick disc, or stellar halo
- Low-mass planets are predicted to be abundant throughout the galaxy, from central regions to the outer halo
- Planet populations differ more strongly around Sun-like stars than around 0.3–0.5 M⊙ dwarfs

## Limitations

- Occurrence rates of low-mass planets remain uncertain, making those findings strongly model-dependent; massive planet results are more robust
- Raw HESTIA snapshots and external NGPPS data must be placed in data/raw and data/external respectively; the README says 'Please contact me for more information' about data access
- Findings are tied to the HESTIA constrained simulation and NGPPS Bern models; other galaxy simulations or synthesis models may yield different component ratios

## Evidence

- [abstract] component aggregation from abstract: "We compared these result with the frequencies in the more distant, metal-poor region such as the thick disc and stellar halo"
- [intro] combine HESTIA and NGPPS: "This project combines the HESTIA suite of high-resolution constrained hydrodynamical galaxy formation simulations with the New Generation Planet Population Synthesis (NGPPS) models from Bern"
- [methods] data sync step: "`make sync_data_from_s3` will use `aws s3 sync` to recursively sync files from `s3://[OPTIONAL] your-bucket-for-syncing-data (do not include 's3://')/data/` to `data/`."
- [readme] galactic decomposition and pipeline entry point: [evidence span withheld — anchor status: external_doc]
- [readme] component labels and planet classes: [evidence span withheld — anchor status: external_doc]
- [readme] data layout: [evidence span withheld — anchor status: external_doc]
