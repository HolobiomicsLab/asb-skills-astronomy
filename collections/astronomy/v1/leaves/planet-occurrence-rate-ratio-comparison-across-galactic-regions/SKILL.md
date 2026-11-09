---
name: planet-occurrence-rate-ratio-comparison-across-galactic-regions
description: Use when occurrence rates of planet types (Earth-like, super-Earths,
  Neptunians, giants) must be stratified by galactic component (thin disc, thick disc,
  bulge, stellar halo) and host-star mass to test whether regional contrast depends
  on stellar metallicity correlations.
when_to_use_negative:
- When only total planet occurrence per region is needed without separating host-star
  mass regimes (the regional contrast differs between Sun-like stars and 0.3–0.5 M⊙
  dwarfs).
- When studying low-mass planet occurrence in isolation and results must be robust
  — the article states low-mass occurrence rates are still uncertain and findings
  are strongly model-dependent.
tools:
- name: NGPPS (New Generation Planet Population Synthesis)
  role: Bern planet population synthesis models that generate occurrence rates for
    Earth-like planets, super-Earths, Neptunians, and giants as a function of host
    mass and metallicity
  repo: https://github.com/ChrisBoettner/skaro
- name: HESTIA galaxy formation simulation
  role: Provides the high-resolution constrained hydrodynamical Milky-Way analogue
    snapshot whose stars are decomposed into bulge, thin disc, thick disc, and stellar
    halo and assigned planets
  repo: https://github.com/ChrisBoettner/skaro
- name: pandas
  role: Data manipulation of occurrence-rate tables and writing the ratio comparison
    CSV
  repo: https://github.com/ChrisBoettner/skaro
- name: yt
  role: Efficient processing and analysis of HESTIA simulation snapshot data when
    loading stellar populations
  repo: https://github.com/ChrisBoettner/skaro
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202449557
    title: Populating the Milky Way
  source_content_hash: sha256:923ccb5e470b574dc3a9d86a7ec738bac0fee2e4df4eb7c77d1b664ea20fb41f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChrisBoettner/skaro
  tools:
  - HESTIA galaxy formation simulation
  - NGPPS (New Generation Planet Population Synthesis)
  - pandas
  - yt
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# planet-occurrence-rate-ratio-comparison-across-galactic-regions

## Summary

Computes thin-disc/thick-disc occurrence-rate ratios for giant planets (>300 M⊕) and low-mass Earth-like planets by galactic region and host-star type, using NGPPS population synthesis applied to a HESTIA Milky-Way analogue, to quantify how strongly planet populations differ across galactic environments.

## When to use

Use when occurrence rates of planet types (Earth-like, super-Earths, Neptunians, giants) must be stratified by galactic component (thin disc, thick disc, bulge, stellar halo) and host-star mass to test whether regional contrast depends on stellar metallicity correlations.

## When NOT to use

- When only total planet occurrence per region is needed without separating host-star mass regimes (the regional contrast differs between Sun-like stars and 0.3–0.5 M⊙ dwarfs).
- When studying low-mass planet occurrence in isolation and results must be robust — the article states low-mass occurrence rates are still uncertain and findings are strongly model-dependent.

## Inputs

- NGPPS occurrence-rate outputs stratified by galactic region (thin disc, thick disc, bulge, stellar halo)
- Host-star type labels (Sun-like stars vs 0.3–0.5 M⊙ dwarfs) with stellar mass and [Fe/H] metallicity from the HESTIA MW analogue snapshot
- Galactic decomposition results (halo/thin disc/thick disc/bulge membership) from data/processed

## Outputs

- occurrence_rate_comparison.csv containing thin-disc/thick-disc occurrence-rate ratios for giant planets (>300 M⊕) and low-mass Earth-like planets per host-star type
- Robustness classification column flagging low-mass planet entries as model-dependent and massive-planet entries as robust

## How to apply

Load occurrence-rate outputs of the population model stratified by galactic region (thin disc, thick disc, bulge, stellar halo) and host type (Sun-like stars vs 0.3–0.5 M⊙ dwarfs), as produced by NGPPS applied to the HESTIA MW analogue. Compute thin-disc/thick-disc occurrence-rate ratios separately for giant planets (>300 M⊕) and low-mass Earth-like planets for each host type. Compare the region-to-region contrast between Sun-like stars and 0.3–0.5 M⊙ dwarfs to quantify that populations differ more strongly around Sun-like stars, reflecting the weaker [Fe/H]–planet-mass correlation there. Classify robustness: mark low-mass planet rates as strongly model-dependent and massive-planet ratios as more robust. Write the ratio table and robustness classification to occurrence_rate_comparison.csv.

## Related tools

- **NGPPS (New Generation Planet Population Synthesis)** (Bern planet population synthesis models that generate occurrence rates for Earth-like planets, super-Earths, Neptunians, and giants as a function of host mass and metallicity) — https://github.com/ChrisBoettner/skaro
- **HESTIA galaxy formation simulation** (Provides the high-resolution constrained hydrodynamical Milky-Way analogue snapshot whose stars are decomposed into bulge, thin disc, thick disc, and stellar halo and assigned planets) — https://github.com/ChrisBoettner/skaro
- **pandas** (Data manipulation of occurrence-rate tables and writing the ratio comparison CSV) — https://github.com/ChrisBoettner/skaro
- **yt** (Efficient processing and analysis of HESTIA simulation snapshot data when loading stellar populations) — https://github.com/ChrisBoettner/skaro

## Examples

```
mamba env create -f environment.yaml; mamba activate skaro; pip install -e .; python -c "from src.skaro.model import Model"
```

## Evaluation signals

- Giant-planet (>300 M⊕) thin-disc/thick-disc ratio falls in the reported 10–20 range.
- Sun-like-star regional contrast (thin/thick disc ratio spread) exceeds that of 0.3–0.5 M⊙ dwarfs.
- Low-mass Earth-like planet entries are flagged 'model-dependent' while massive-planet entries are flagged 'robust' in the output table.
- Low-mass Earth-like planets around Sun-like stars show a thick-disc/thin-disc ratio of ~1.5 (thick disc more frequent).
- occurrence_rate_comparison.csv contains ratio rows for both host types and both planet mass classes with a robustness column.

## Limitations

- Occurrence rates of low-mass planets remain uncertain, making findings strongly model-dependent; only massive-planet ratios are robust.
- Low-mass planets form with weaker metallicity dependence, so their regional contrast is small compared to giant planets.
- Results apply to a single HESTIA Milky-Way analogue; raw HESTIA snapshots and NGPPS external data must be supplied separately (contact the author for data access).

## Evidence

- [abstract] Giant planet >300 M⊕ thin/thick disc ratio 10–20: "The occurrence rate of giant planets (>300 M ⊕ ) is 10–20 times larger in the thin disc compared to the thick disc"
- [abstract] Regional contrast stronger for Sun-like hosts: "planet populations differ more strongly around Sun-like stars compared to dwarfs with masses 0.3–0.5 M ⊙"
- [abstract] Robustness split between planet mass classes: "the occurrence rates of low-mass planets are still uncertain, making our findings strongly model-dependent. Massive planets are more comprehensively understood and our findings are more robust"
- [abstract] Low-mass Earth-like planets thick-disc enrichment: "low-mass Earth-like planets around Sun-like stars are most abundant in the thick disc, being 1.5 times more frequent than in the thin disc"
- [readme] Integration of HESTIA and NGPPS: [evidence span withheld — anchor status: external_doc]
- [readme] Galactic decomposition into components: [evidence span withheld — anchor status: external_doc]
