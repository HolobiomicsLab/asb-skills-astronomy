---
name: neutron-star-radius-measurement-extraction
description: Use when a paper reports a 1.4M⊙ neutron-star radius with asymmetric
  credible intervals (e.g., R = 11.98 +0.35/−0.40 km) and you must benchmark it against
  prior literature values to verify a claimed precision improvement.
when_to_use_negative:
- The paper reports only a single radius point estimate with symmetric errors and
  no credible-interval table — there is nothing to cross-compare widths against.
- The target is measuring a radius from raw GW/kilonova/GRB data with NMMA itself
  (run nmma-generation/nmma-analysis) rather than extracting and benchmarking already-published
  values.
- The quantity is not R_1.4 — e.g., tidal deformability, EOS constraints, or the Hubble
  constant — since Table 1 entries and widths are defined specifically for the 1.4M⊙
  radius.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: NMMA
  role: 'Framework that produced the ''this work'' Table 1 entry: joint Bayesian analysis
    of GW170817, AT2017gfo, and GRB170817A yielding R = 11.98 (+0.35/−0.40) km; backend
    is bilby with dynesty/PyMultiNest/UltraNest samplers'
  repo: https://github.com/nuclear-multimessenger-astronomy/nmma
- name: pandas
  role: Build the radius-comparison DataFrame and export it as radius_comparison_table.csv
    with computed credible-interval width column
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-43932-6
    title: An updated nuclear-physics and multi-messenger astrophysics framework for
      binary neutron star mergers
  source_content_hash: sha256:3e9fd627533e604aa4f5391218abb32b2257e0234dc49ec894a3c32c691af69d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nuclear-multimessenger-astronomy/nmma
  tools:
  - NMMA
  - pandas
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# neutron-star-radius-measurement-extraction

## Summary

Extracts and cross-compares credible-interval estimates of the 1.4 solar-mass neutron-star radius from Table 1 of the Pang et al. 2023 Nature Communications NMMA paper and prior literature, quantifying how joint GW (GW170817), kilonova (AT2017gfo), and GRB afterglow (GRB170817A) analysis narrows the R_1.4 constraint.

## When to use

Use when a paper reports a 1.4M⊙ neutron-star radius with asymmetric credible intervals (e.g., R = 11.98 +0.35/−0.40 km) and you must benchmark it against prior literature values to verify a claimed precision improvement.

## When NOT to use

- The paper reports only a single radius point estimate with symmetric errors and no credible-interval table — there is nothing to cross-compare widths against.
- The target is measuring a radius from raw GW/kilonova/GRB data with NMMA itself (run nmma-generation/nmma-analysis) rather than extracting and benchmarking already-published values.
- The quantity is not R_1.4 — e.g., tidal deformability, EOS constraints, or the Hubble constant — since Table 1 entries and widths are defined specifically for the 1.4M⊙ radius.

## Inputs

- Table 1 of Pang et al. 2023 (Nature Communications, DOI 10.1038/s41467-023-43932-6): per-reference R_1.4 central values and credible-interval bounds
- Eight literature entries: Dietrich 2020, Essick 2020, Breschi 2022, Nicholl 2022, Raaijmakers 2021, Miller 2021, Huth 2022, this work (NMMA joint analysis)

## Outputs

- radius_comparison_table.csv with columns: reference, radius central value (km), lower credible bound, upper credible bound, credible-interval width
- Validated 'this work' record: R = 11.98 km, +0.35/−0.40 km, width ≈ 0.75 km
- Comparison verdict on whether the joint NMMA width is visibly narrower than prior literature entries

## How to apply

Read Table 1 and extract, for each of the eight entries (Dietrich 2020, Essick 2020, Breschi 2022, Nicholl 2022, Raaijmakers 2021, Miller 2021, Huth 2022, this work), the central R_1.4 value plus lower and upper credible-interval bounds. Assemble these into a pandas DataFrame with columns for reference, central radius, lower and upper bounds, and computed interval width (upper minus lower), then write to radius_comparison_table.csv. Validate that the 'this work' row reproduces R = 11.98 (+0.35/−0.40) km — a total width of ≈0.75 km — and confirm that this width is noticeably narrower than the majority of the seven prior entries, which is the paper's evidence that simultaneously analyzing the gravitational-wave signal, the kilonova, and the gamma-ray burst afterglow improves the constraint. The 'this work' entry corresponds to the NMMA joint analysis of GW170817, AT2017gfo, and GRB170817A (GRB afterglow modeled with a synthetic Gaussian jet-model lightcurve and the Troja et al. data set).

## Related tools

- **NMMA** (Framework that produced the 'this work' Table 1 entry: joint Bayesian analysis of GW170817, AT2017gfo, and GRB170817A yielding R = 11.98 (+0.35/−0.40) km; backend is bilby with dynesty/PyMultiNest/UltraNest samplers) — https://github.com/nuclear-multimessenger-astronomy/nmma
- **pandas** (Build the radius-comparison DataFrame and export it as radius_comparison_table.csv with computed credible-interval width column)

## Examples

```
import pandas as pd; df = pd.DataFrame([{'reference':'this work','R_km':11.98,'lower':-0.40,'upper':0.35}]); df['width_km'] = df['upper']-df['lower']; df.to_csv('radius_comparison_table.csv', index=False)
```

## Evaluation signals

- The 'this work' row must reproduce exactly R = 11.98 km with +0.35/−0.40 km bounds from Table 1.
- Computed credible-interval width for 'this work' equals upper minus lower bound ≈ 0.75 km.
- The 'this work' width must be visibly (noticeably) narrower than the majority of the seven prior literature entries, consistent with the paper's improvement claim.
- radius_comparison_table.csv must contain all eight references with non-negative widths and correctly oriented asymmetric bounds (lower below, upper above the central value).

## Limitations

- Asymmetric credible intervals must be tracked as separate lower/upper bounds; collapsing them to a single symmetric error would misstate widths.
- Table 1 width comparison reflects credible intervals, not frequentist confidence intervals; widths across references may rest on different data subsets and priors.
- The joint NMMA analysis itself carries additional uncertainties from limited physics input in kilonova models and semi-analytic GRB models.

## Evidence

- [abstract] Joint NMMA R_1.4 estimate: [evidence span withheld — anchor status: not_found]
- [intro] Multi-messenger dataset: "The study of the gravitational-wave (GW) and electromagnetic (EM) signals GW1708171, AT2017gfo2–12, and GRB170817A13–15 has already enabled numerous scientific breakthroughs"
- [results] GRB afterglow modeling: "The GRB afterglow light-curve data are analyzed with the synthetic Gaussian jet-model lightcurve described before57,58"
- [results] Kilonova and afterglow fit quality: [evidence span withheld — anchor status: not_found]
- [readme] NMMA citation and backend: "kindly make a reference to the package by its name, NMMA, and a citation to the companion paper"
