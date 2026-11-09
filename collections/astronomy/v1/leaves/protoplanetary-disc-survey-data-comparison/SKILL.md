---
name: protoplanetary-disc-survey-data-comparison
description: Use when an ALMA disc survey catalog of 12CO integrated fluxes and CO-disc
  radii must be overlaid with semi-analytical model tracks to test agreement and quantify
  flux-based vs size-based sample increases.
when_to_use_negative:
- Input is a resolved, disc-by-disc thermochemical study where computationally-expensive
  physico-chemical models (e.g. DALI) are feasible and higher fidelity is required
  — the semi-analytical method is only accurate within a factor of three
- 'The science question is the underlying disc evolution mechanism itself (viscosity
  vs MHD winds): the available ALMA data have not given conclusive answers and uncertainties
  on initial conditions limit interpretation'
- The survey lacks both 12CO integrated fluxes and CO-disc radii, so the flux-size
  plane comparison cannot be constructed
edam_operation: http://edamontology.org/operation_2428
tools:
- name: ALMA
  role: Source of the survey observations (12CO integrated fluxes and disc sizes)
    for Lupus and Upper Sco against which model tracks are compared
- name: COpops
  role: Semi-analytical 12CO flux model used to compute predicted fluxes (optically
    thick assumption, optional CO depletion) for each disc
  repo: github.com/fzagaria/COpops
- name: DALI
  role: 'Thermochemical benchmark: the semi-analytical fluxes must agree with DALI
    within a factor of three, setting the acceptance band used in the model–data comparison'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202346164
    title: Testing protoplanetary disc evolution with CO fluxes
  source_content_hash: sha256:9ef1d2243f45dc180b2942b65a7d062d71c88c8eb35565eca863ebd584d95351
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALMA
  - COpops
  - DALI
  edam_operation: http://edamontology.org/operation_2428
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# protoplanetary-disc-survey-data-comparison

## Summary

Compares semi-analytical 12CO flux models (with and without a CO-depletion prescription) against ALMA disc-survey observations in the flux-size plane, quantifying model–data agreement and the sample-size gain of selecting discs by flux rather than size.

## When to use

Use when an ALMA disc survey catalog of 12CO integrated fluxes and CO-disc radii must be overlaid with semi-analytical model tracks to test agreement and quantify flux-based vs size-based sample increases.

## When NOT to use

- Input is a resolved, disc-by-disc thermochemical study where computationally-expensive physico-chemical models (e.g. DALI) are feasible and higher fidelity is required — the semi-analytical method is only accurate within a factor of three
- The science question is the underlying disc evolution mechanism itself (viscosity vs MHD winds): the available ALMA data have not given conclusive answers and uncertainties on initial conditions limit interpretation
- The survey lacks both 12CO integrated fluxes and CO-disc radii, so the flux-size plane comparison cannot be constructed

## Inputs

- Survey catalog of 12CO integrated fluxes for disc samples (e.g. Upper Sco, Lupus)
- CO-disc radii from the same ALMA survey
- Semi-analytical 12CO flux model implementation (COpops), run with and without CO depletion

## Outputs

- Per-disc and sample-level model–data comparison table (e.g. upper_sco_model_comparison.csv)
- Flux-size comparison plot with model tracks over observations (e.g. upper_sco_flux_size_comparison.png)
- Sample-size ratio: discs selected by 12CO flux vs by disc size (expected factor 3.6 for Upper Sco)

## How to apply

Load the survey catalog of 12CO integrated fluxes and CO-disc radii (e.g. the Upper Sco sample) as the observational reference. For each disc, compute predicted 12CO fluxes with the semi-analytical model implemented in COpops, which assumes optically thick 12CO so that fluxes scale with disc surface area. Repeat the computation with a CO-depletion prescription switched on, reducing the gas-phase CO abundance below the canonical value. Overlay both model tracks on the observed flux-size plane and test agreement quantitatively using the paper's factor-of-three tolerance (the same band achieved when benchmarking against DALI thermochemical models); agreement is expected only for the CO-depleted models. Then count discs detected/usable when selected by 12CO flux versus by disc size, verifying the reported sample increase (factor 3.6 for Upper Sco, 1.3 for Lupus). Write per-disc and sample-level comparisons to a CSV (e.g. upper_sco_model_comparison.csv) and save the flux-size comparison figure (e.g. upper_sco_flux_size_comparison.png).

## Related tools

- **ALMA** (Source of the survey observations (12CO integrated fluxes and disc sizes) for Lupus and Upper Sco against which model tracks are compared)
- **COpops** (Semi-analytical 12CO flux model used to compute predicted fluxes (optically thick assumption, optional CO depletion) for each disc) — github.com/fzagaria/COpops
- **DALI** (Thermochemical benchmark: the semi-analytical fluxes must agree with DALI within a factor of three, setting the acceptance band used in the model–data comparison)

## Examples

```
python upper_sco_model_comparison.py --catalog upper_sco_catalog.csv --co-depletion --tolerance 3 --output upper_sco_model_comparison.csv --plot upper_sco_flux_size_comparison.png
```

## Evaluation signals

- Model tracks agree with Upper Sco observations only when the CO-depletion prescription is enabled; non-depleted models fall outside the acceptance band
- Agreement between semi-analytical fluxes and DALI benchmarks stays within a factor of three
- Flux-selected sample is larger than the size-selected sample by a factor of 3.6 for Upper Sco (1.3 for Lupus)
- Output CSV contains per-disc flux predictions plus a sample-level comparison row; the flux-size figure shows both CO-depleted and canonical-abundance tracks overlaid on the observations

## Limitations

- Uncertainties on initial conditions limited the interpretation of the observations
- Accuracy is bounded: the semi-analytical model recovers only within-a-factor-of-three agreement with full thermochemical models
- The available ALMA data have not given conclusive answers on the underlying disc evolution mechanisms (viscosity vs MHD winds)

## Evidence

- [abstract] Agreement requires CO depletion: "Models and data agree well only if CO depletion is considered"
- [abstract] DALI factor-of-three benchmark: "benchmarked it against the results of DALI thermochemical models, recovering an agreement within a factor of three"
- [abstract] Sample increase factor: "increased samples, by a factor 1.3 (Lupus) and 3.6 (Upper Sco), when studying fluxes instead of sizes"
- [abstract] Optically thick flux scaling: "assuming that 12CO is optically thick, fluxes scale with the disc surface area"
- [abstract] Initial-condition uncertainties: "the uncertainties on the initial conditions limited our interpretation of the observations"
- [readme] COpops scope: "This method is fast, flexible and accurate: compared to more-complex physico-chemical models, we recover a good agreement within a factor of three."
