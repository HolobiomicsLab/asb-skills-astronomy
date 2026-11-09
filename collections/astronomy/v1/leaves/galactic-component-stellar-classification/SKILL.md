---
name: galactic-component-stellar-classification
description: Use when the input is a HESTIA (or comparable) hydrodynamical MW-analogue
  snapshot with per-star metallicity, age, and mass, and the analysis goal requires
  grouping stars by Galactic component before computing planet occurrence rates.
when_to_use_negative:
- Input is an observed stellar catalog rather than a HESTIA/simulated MW-analogue
  snapshot — the decomposition relies on simulated halo and particle structure
- The analysis does not stratify by Galactic component (e.g. galaxy-integrated planet
  counts), since the classification adds no information there
tools:
- name: HESTIA galaxy formation simulation
  role: Provides the simulated MW-analogue snapshots whose stellar populations are
    decomposed into Galactic components
  repo: github.com/ChrisBoettner/skaro
- name: yt
  role: Efficient loading and processing of the simulation snapshot data during decomposition
  repo: github.com/ChrisBoettner/skaro
- name: pandas
  role: Data manipulation of per-star fields (metallicity, age, component assignment)
  repo: github.com/ChrisBoettner/skaro
- name: sklearn
  role: Machine learning utilities used in the skaro analysis pipeline
  repo: github.com/ChrisBoettner/skaro
- name: Makefile / aws s3 sync
  role: Retrieves raw HESTIA snapshots from S3 into `data/raw` before decomposition
  repo: github.com/ChrisBoettner/skaro
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202449557
    title: Populating the Milky Way
  source_content_hash: sha256:5377a37af761d5963ebfefc8b09cd0490acce8ea920e5d611522ac1826bd12b2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HESTIA galaxy formation simulation
  - Makefile
  - Makefile / aws s3 sync
  - aws s3 sync
  - pandas
  - sklearn
  - yt
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# galactic-component-stellar-classification

## Summary

Assigns each star in a HESTIA Milky Way-analogue simulation snapshot to a Galactic component — bulge, thin disc, thick disc, or stellar halo — using the galaxy decomposition implemented in the skaro project, so that planet occurrence rates can be aggregated per stellar environment.

## When to use

Use when the input is a HESTIA (or comparable) hydrodynamical MW-analogue snapshot with per-star metallicity, age, and mass, and the analysis goal requires grouping stars by Galactic component before computing planet occurrence rates.

## When NOT to use

- Input is an observed stellar catalog rather than a HESTIA/simulated MW-analogue snapshot — the decomposition relies on simulated halo and particle structure
- The analysis does not stratify by Galactic component (e.g. galaxy-integrated planet counts), since the classification adds no information there

## Inputs

- HESTIA simulation snapshot files in `data/raw` (per-star particle data with metallicity, age, mass, positions)
- NGPPS planet synthesis data, stellar parameter data, and HZ coefficients in `data/external`

## Outputs

- Per-star Galactic component assignments (bulge / thin disk / thick disk / halo)
- Halo information and galaxy decomposition results stored in `data/processed`

## How to apply

Retrieve the raw HESTIA snapshots via `make sync_data_from_s3` (aws s3 sync into `data/raw`), then use the `Model` class in `src/skaro/model.py` to load the simulation snapshot, create the halo object, add stellar fields, and run the galaxy decomposition. Each star carries its metallicity, age, mass, and Galactic component assignment; the decomposition splits the galaxy into its major components (bulge, thin disk, thick disk, and halo) so that occurrence rates of giant planets (>300 M⊕), Earth-like planets, and low-mass planets can be aggregated separately per component. Processed halo information and decomposition results are written to `data/processed` for downstream NGPPS-based planet assignment. Rationale: because metal-rich components (bulge, thin disc) and metal-poor components (thick disc, stellar halo) host very different planet demographics — e.g. 10–20× more giants in the thin disc — the component classification is the essential stratification variable.

## Related tools

- **HESTIA galaxy formation simulation** (Provides the simulated MW-analogue snapshots whose stellar populations are decomposed into Galactic components) — github.com/ChrisBoettner/skaro
- **yt** (Efficient loading and processing of the simulation snapshot data during decomposition) — github.com/ChrisBoettner/skaro
- **pandas** (Data manipulation of per-star fields (metallicity, age, component assignment)) — github.com/ChrisBoettner/skaro
- **sklearn** (Machine learning utilities used in the skaro analysis pipeline) — github.com/ChrisBoettner/skaro
- **Makefile / aws s3 sync** (Retrieves raw HESTIA snapshots from S3 into `data/raw` before decomposition) — github.com/ChrisBoettner/skaro

## Examples

```
make sync_data_from_s3
```

## Evaluation signals

- Decomposition output assigns every star in the halo object to exactly one of the four components: bulge, thin disk, thick disk, or halo
- Component assignment preserves the expected metallicity ordering: bulge and thin disc stars are metal-rich; thick disc and stellar halo stars are metal-poor
- Processed halo information and galaxy decomposition results exist in `data/processed` before downstream planet occurrence aggregation runs
- Downstream validation: giant planet (>300 M⊕) occurrence is 10–20× larger in the thin disc than the thick disc, confirming components were stratified correctly

## Limitations

- Occurrence rates of low-mass planets remain uncertain, making component-level results for low-mass planets strongly model-dependent; massive-planet results are more robust
- Raw HESTIA snapshots are not shipped in the repository — data must be obtained separately (contact the authors); `make sync_data_from_s3` requires a configured S3 bucket
- Planet populations differ more strongly around Sun-like stars than around 0.3–0.5 M⊙ dwarfs, so component-level conclusions are host-mass dependent

## Evidence

- [readme] Galactic Decomposition feature: [evidence span withheld — anchor status: external_doc]
- [readme] Model class responsibilities: [evidence span withheld — anchor status: external_doc]
- [abstract] Comparison across components: "We compared these result with the frequencies in the more distant, metal-poor region such as the thick disc and stellar halo"
- [readme] Data layout: [evidence span withheld — anchor status: external_doc]
