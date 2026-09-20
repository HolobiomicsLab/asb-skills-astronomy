---
name: stellar-mass-dependent-planet-demographics-analysis
description: Use when comparing planet occurrence rates across galactic components
  (thin disc, thick disc, bulge, stellar halo) stratified by host type (Sun-like stars
  vs 0.3–0.5 M⊙ dwarfs), or when assessing whether metallicity ([Fe/H]) drives regional
  differences in planet populations.
when_to_use_negative:
- The input is raw HESTIA simulation snapshot data that has not yet been processed
  into halo information and galaxy decomposition results via the Model class
- The analysis concerns a single planet type within one galactic region and does not
  require cross-region or cross-host-mass comparison
- The host stars are outside the regimes examined here (Sun-like or 0.3–0.5 M⊙ dwarfs),
  e.g. purely high-mass stellar populations
edam_operation: http://edamontology.org/operation_2409
tools:
- name: NGPPS (New Generation Planet Population Synthesis)
  role: Bern planet population synthesis models used to generate planet populations
    for host stars of given mass and metallicity, the basis of the occurrence-rate
    outputs
- name: HESTIA galaxy formation simulation
  role: Provides the simulated Milky-Way analogue whose stellar populations are decomposed
    into bulge, thin disk, thick disk, and halo for regional stratification
- name: skaro (ChrisBoettner/skaro)
  role: Model class loads the simulation snapshot, creates the halo object, adds stellar
    and planet fields, and performs the galaxy decomposition feeding the occurrence-rate
    comparison
  repo: github.com/ChrisBoettner/skaro
- name: yt
  role: Efficient processing and analysis of HESTIA simulation snapshot data
  repo: github.com/ChrisBoettner/skaro
- name: pandas
  role: Data manipulation for occurrence-rate tables and ratio computation
  repo: github.com/ChrisBoettner/skaro
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202449557
    title: Populating the Milky Way
  source_content_hash: sha256:e9184eeba01bfd8774198988f2d6023f68bbd360b2cd8d469bae27751897f612
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HESTIA galaxy formation simulation
  - NGPPS (New Generation Planet Population Synthesis)
  - pandas
  - skaro (ChrisBoettner/skaro)
  - yt
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# stellar-mass-dependent-planet-demographics-analysis

## Summary

A skill for quantifying how galactic-region planet occurrence-rate contrasts (thin disc vs thick disc) depend on host-star mass, using NGPPS planet population synthesis applied to a HESTIA Milky-Way analogue. It produces stratified occurrence-rate ratio tables with robustness flags for massive versus low-mass planets.

## When to use

Use when comparing planet occurrence rates across galactic components (thin disc, thick disc, bulge, stellar halo) stratified by host type (Sun-like stars vs 0.3–0.5 M⊙ dwarfs), or when assessing whether metallicity ([Fe/H]) drives regional differences in planet populations.

## When NOT to use

- The input is raw HESTIA simulation snapshot data that has not yet been processed into halo information and galaxy decomposition results via the Model class
- The analysis concerns a single planet type within one galactic region and does not require cross-region or cross-host-mass comparison
- The host stars are outside the regimes examined here (Sun-like or 0.3–0.5 M⊙ dwarfs), e.g. purely high-mass stellar populations

## Inputs

- Occurrence-rate outputs from the NGPPS population model, stratified by galactic region (thin disc, thick disc, bulge, stellar halo)
- Host-star mass and metallicity ([Fe/H]) fields from HESTIA simulation snapshots (loaded via the Model class in src/skaro/model.py)
- Galactic decomposition results (bulge, thin disk, thick disk, halo assignments) stored in data/processed
- NGPPS data, stellar parameter data, and HZ coefficients from data/external

## Outputs

- Ratio table of thin-disc/thick-disc occurrence rates per planet type and host mass regime (e.g. occurrence_rate_comparison.csv)
- Robustness classification: low-mass planet entries flagged as strongly model-dependent, massive-planet entries flagged as robust
- Summary metrics such as the giant-planet (>300 M⊕) thin-disc/thick-disc ratio (expected in the 10–20 range)

## How to apply

Load the occurrence-rate outputs of the NGPPS population model computed on the HESTIA Milky-Way analogue, stratified by galactic region (thin disc, thick disc, bulge, stellar halo) and stellar host type. Compute thin-disc/thick-disc occurrence-rate ratios separately for giant planets (>300 M⊕) and low-mass Earth-like planets for each host type. Compare the region-to-region contrast between Sun-like stars and 0.3–0.5 M⊙ dwarfs: Sun-like stars should show stronger regional differences because their planet formation has a weaker [Fe/H]-planet-mass correlation. Classify robustness by contrasting the model-dependent variability of low-mass planet rates against the more robust massive-planet ratios, then write the ratio table and classification (e.g. to occurrence_rate_comparison.csv).

## Related tools

- **NGPPS (New Generation Planet Population Synthesis)** (Bern planet population synthesis models used to generate planet populations for host stars of given mass and metallicity, the basis of the occurrence-rate outputs)
- **HESTIA galaxy formation simulation** (Provides the simulated Milky-Way analogue whose stellar populations are decomposed into bulge, thin disk, thick disk, and halo for regional stratification)
- **skaro (ChrisBoettner/skaro)** (Model class loads the simulation snapshot, creates the halo object, adds stellar and planet fields, and performs the galaxy decomposition feeding the occurrence-rate comparison) — github.com/ChrisBoettner/skaro
- **yt** (Efficient processing and analysis of HESTIA simulation snapshot data) — github.com/ChrisBoettner/skaro
- **pandas** (Data manipulation for occurrence-rate tables and ratio computation) — github.com/ChrisBoettner/skaro

## Examples

```
mamba env create -f environment.yaml && pip install -e . && python -c "from skaro.model import Model; m = Model()"
```

## Evaluation signals

- Giant-planet (>300 M⊕) thin-disc/thick-disc occurrence-rate ratio falls in the reported 10–20 range
- The region-to-region contrast around Sun-like stars exceeds that around 0.3–0.5 M⊙ dwarfs
- Low-mass Earth-like planets around Sun-like stars show the expected thick-disc/thin-disc ratio (~1.5, i.e. 1.5 times more frequent in the thick disc)
- Low-mass planet occurrence-rate entries are flagged as strongly model-dependent while massive-planet entries are flagged as robust
- Ratio table is written to occurrence_rate_comparison.csv with one row per planet type × host type × region pair

## Limitations

- Occurrence rates of low-mass planets are still uncertain, making the findings strongly model-dependent; massive planets are more comprehensively understood and those findings are more robust
- Low-mass Earth-like planet results can run counter to metallicity intuition (thick disc ~1.5× thin disc), so ratios must not be interpreted as monotonic functions of [Fe/H]
- Raw HESTIA snapshots must be pre-processed via the skaro Model class (halo object, stellar and planet fields, galaxy decomposition) before ratio computation; raw data access requires contacting the authors

## Evidence

- [abstract] giant vs low-mass ratios: "The occurrence rate of giant planets (>300 M ⊕ ) is 10–20 times larger in the thin disc compared to the thick disc"
- [abstract] host-mass dependence: "planet populations differ more strongly around Sun-like stars compared to dwarfs with masses 0.3–0.5 M ⊙"
- [abstract] low-mass counterintuitive result: "low-mass Earth-like planets around Sun-like stars are most abundant in the thick disc, being 1.5 times more frequent than in the thin disc"
- [abstract] robustness classification rationale: "the occurrence rates of low-mass planets are still uncertain, making our findings strongly model-dependent. Massive planets are more comprehensively understood and our findings are more robust"
- [readme] pipeline entry point: [evidence span withheld — anchor status: external_doc]
