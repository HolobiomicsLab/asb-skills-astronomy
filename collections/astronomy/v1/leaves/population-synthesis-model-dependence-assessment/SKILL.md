---
name: population-synthesis-model-dependence-assessment
description: Use when occurrence-rate ratios from a planet population synthesis (e.g.
  thin/thick-disc ratios per stellar host type) must be classified as robust or model-dependent
  before being reported.
when_to_use_negative:
- The goal is absolute planet occurrence rates rather than assessing how model-dependent
  comparative regional ratios are
- The input population does not distinguish planet-mass classes (e.g. no >300 M⊕ giant
  vs low-mass Earth-like split) or galactic components, so no robustness contrast
  can be made
- Low-mass planet occurrence rates have been independently validated — the model-dependence
  flag would no longer apply
tools:
- name: NGPPS (New Generation Planet Population Synthesis)
  role: Bern planet population synthesis model that generates the planet occurrence
    rates whose model dependence is being assessed
- name: HESTIA galaxy formation simulation
  role: Provides the simulated Milky Way analogue whose stellar populations (mass,
    metallicity) and galactic decomposition feed the occurrence-rate ratios
- name: skaro
  role: Repository implementing the planet assignment model, galaxy decomposition
    (Model class in src/skaro/model.py), and occurrence-rate analysis used for the
    assessment
  repo: github.com/ChrisBoettner/skaro
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202449557
    title: Populating the Milky Way
  source_content_hash: sha256:7212f51e6299c423a306491b8e627aadbdd0d8a4fe44b8cbe9cd9f9b2f15e12d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HESTIA galaxy formation simulation
  - NGPPS (New Generation Planet Population Synthesis)
  - skaro
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# population-synthesis-model-dependence-assessment

## Summary

Quantifies how strongly planet-occurrence-rate conclusions from the NGPPS/HESTIA population model depend on the synthesis model itself, by contrasting robust massive-planet (>300 M⊕) regional ratios against model-dependent low-mass planet rates.

## When to use

Use when occurrence-rate ratios from a planet population synthesis (e.g. thin/thick-disc ratios per stellar host type) must be classified as robust or model-dependent before being reported.

## When NOT to use

- The goal is absolute planet occurrence rates rather than assessing how model-dependent comparative regional ratios are
- The input population does not distinguish planet-mass classes (e.g. no >300 M⊕ giant vs low-mass Earth-like split) or galactic components, so no robustness contrast can be made
- Low-mass planet occurrence rates have been independently validated — the model-dependence flag would no longer apply

## Inputs

- Occurrence-rate outputs from the NGPPS population model, stratified by galactic region (thin disc, thick disc, bulge, stellar halo) and host type (Sun-like vs 0.3–0.5 M⊙ dwarfs)
- Planet classification by mass: giant planets (>300 M⊕) vs low-mass Earth-like planets
- Galaxy decomposition of the HESTIA MW analogue into bulge, thin disc, thick disc, and halo

## Outputs

- Ratio table (thin-disc/thick-disc occurrence-rate ratios) per planet-mass class and stellar host type
- Robustness classification per entry (robust for massive planets, model-dependent for low-mass planets), written to occurrence_rate_comparison.csv

## How to apply

Compute occurrence-rate ratios between galactic regions (thin disc, thick disc, bulge, stellar halo) separately for giant planets (>300 M⊕) and low-mass Earth-like planets, using the NGPPS planet population synthesis applied to the HESTIA Milky-Way analogue. Stratify by stellar host type (Sun-like stars vs 0.3–0.5 M⊙ dwarfs) and verify that the Sun-like-star regional contrast exceeds the dwarf contrast, reflecting the weaker [Fe/H]–planet-mass correlation for low-mass planet formation around dwarfs. Check the giant-planet thin-disc/thick-disc ratio against the reported 10–20 range as a calibration invariant. Then classify robustness: massive-planet ratios are marked robust because massive planets are comprehensively understood, while low-mass-planet rates (e.g. the 1.5× thick-disc over thin-disc excess of Earth-like planets around Sun-like stars) are flagged strongly model-dependent due to uncertain low-mass occurrence rates. Write the ratio table with its robustness classification to occurrence_rate_comparison.csv.

## Related tools

- **NGPPS (New Generation Planet Population Synthesis)** (Bern planet population synthesis model that generates the planet occurrence rates whose model dependence is being assessed)
- **HESTIA galaxy formation simulation** (Provides the simulated Milky Way analogue whose stellar populations (mass, metallicity) and galactic decomposition feed the occurrence-rate ratios)
- **skaro** (Repository implementing the planet assignment model, galaxy decomposition (Model class in src/skaro/model.py), and occurrence-rate analysis used for the assessment) — github.com/ChrisBoettner/skaro

## Examples

```
mamba env create -f environment.yaml && mamba activate skaro && pip install -e . && python -c "from skaro.model import Model"
```

## Evaluation signals

- Giant-planet (>300 M⊕) thin-disc/thick-disc occurrence-rate ratio falls in the reported 10–20 range
- Regional (thin/thick disc) contrast around Sun-like stars exceeds that around 0.3–0.5 M⊙ dwarfs
- Low-mass planet occurrence rates are flagged as strongly model-dependent; massive-planet entries are flagged robust
- Low-mass Earth-like planets around Sun-like stars reproduce the ~1.5× thick-disc/thin-disc frequency ratio
- Output table occurrence_rate_comparison.csv contains both the ratio values and the per-entry robustness classification

## Limitations

- Occurrence rates of low-mass planets remain uncertain, making low-mass findings strongly model-dependent
- Massive-planet findings are more robust only because massive planets are more comprehensively understood; robustness does not extend to low-mass planet conclusions
- Results assume the HESTIA Milky-Way analogue and NGPPS framework; raw HESTIA snapshots and NGPPS data must be supplied externally (data/raw, data/external)

## Evidence

- [abstract] the occurrence rates of low-mass planets are still uncertain, making our findings strongly model-dependent. Massive planets are more comprehensively understood and our findings are more robust: "the occurrence rates of low-mass planets are still uncertain, making our findings strongly model-dependent. Massive planets are more comprehensively understood and our findings are more robust"
- [abstract] The occurrence rate of giant planets (>300 M ⊕ ) is 10–20 times larger in the thin disc compared to the thick disc: "The occurrence rate of giant planets (>300 M ⊕ ) is 10–20 times larger in the thin disc compared to the thick disc"
- [readme] Galactic Decomposition: Divides the simulated galaxy into its major components (bulge, thin disk, thick disk, and halo) to analyze planet demographics in distinct stellar environments.: [evidence span withheld — anchor status: external_doc]
- [readme] The main entry point for running the code is the `Model` class in `src/skaro/model.py`.: [evidence span withheld — anchor status: external_doc]
