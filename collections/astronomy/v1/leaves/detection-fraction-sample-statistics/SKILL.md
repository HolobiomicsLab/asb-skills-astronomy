---
name: detection-fraction-sample-statistics
description: Use when comparing survey detection counts for two observables of the
  same disc sample (e.g. 12CO flux vs disc size) to decide which observable maximizes
  the usable population for model–data comparison.
when_to_use_negative:
- When interpreting individual disc structures or kinematics — the skill applies only
  to population-level sample statistics and flux-size scaling
- When the observable is continuum dust flux or dust disc size rather than 12CO flux/size;
  the reported factors (3.6, 1.3) are specific to CO observables
- When a computationally-expensive thermochemical treatment (e.g. DALI) is required
  per disc; the semi-analytical model is a fast alternative accurate only within a
  factor of three
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: ALMA
  role: Provided the low/intermediate-resolution large disc samples of 12CO integrated
    fluxes and CO-disc radii used as the observational input
- name: COpops
  role: Repository implementing the semi-analytical 12CO flux model used to generate
    model tracks for comparison with Upper Sco and Lupus data
  repo: https://github.com/fzagaria/COpops
- name: DALI
  role: Thermochemical models against which the semi-analytical flux model was benchmarked,
    recovering agreement within a factor of three
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202346164
    title: Testing protoplanetary disc evolution with CO fluxes
  source_content_hash: sha256:4ea7e4c823cfcdbdf68e702998f14c44acb833b5d12ffe538b1744e04772ab2a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fzagaria/COpops
  tools:
  - ALMA
  - COpops
  - DALI
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# detection-fraction-sample-statistics

## Summary

Quantifies how much larger a usable disc sample is when ALMA 12CO integrated fluxes, rather than CO-disc radii (sizes), are used as the selection observable. In the Zagaria et al. (2023) study this yielded sample increases of a factor 1.3 (Lupus) and 3.6 (Upper Sco).

## When to use

Use when comparing survey detection counts for two observables of the same disc sample (e.g. 12CO flux vs disc size) to decide which observable maximizes the usable population for model–data comparison.

## When NOT to use

- When interpreting individual disc structures or kinematics — the skill applies only to population-level sample statistics and flux-size scaling
- When the observable is continuum dust flux or dust disc size rather than 12CO flux/size; the reported factors (3.6, 1.3) are specific to CO observables
- When a computationally-expensive thermochemical treatment (e.g. DALI) is required per disc; the semi-analytical model is a fast alternative accurate only within a factor of three

## Inputs

- Survey catalog of 12CO integrated fluxes (e.g. Upper Sco ALMA survey)
- Catalog of CO-disc radii (sizes) for the same discs
- Semi-analytical 12CO flux model tracks from COpops (with and without CO depletion)

## Outputs

- Detection-fraction / sample-size ratio per region (e.g. factor 3.6 for Upper Sco, 1.3 for Lupus)
- Per-disc and sample-level model–data comparison table (e.g. upper_sco_model_comparison.csv)
- Flux-size comparison plot with overlaid model tracks (e.g. upper_sco_flux_size_comparison.png)

## How to apply

Load the survey catalog of 12CO integrated fluxes and CO-disc radii for each star-forming region (Upper Sco, Lupus). Count discs detected/usable when selected on 12CO flux and separately when selected on disc size, then compute the ratio of sample sizes (reported as 3.6 for Upper Sco and 1.3 for Lupus). For the model comparison, compute predicted 12CO fluxes with the semi-analytical model in the COpops repository, which assumes optically thick 12CO emission so fluxes scale with disc surface area, and repeat with a CO-depletion prescription switched on. Overlay model tracks on the observed flux-size plane and accept agreement only within the paper's factor-of-three tolerance — agreement holds only when CO depletion is included. Write per-disc and sample-level results to a comparison table (e.g. upper_sco_model_comparison.csv) and the flux-size plot (e.g. upper_sco_flux_size_comparison.png).

## Related tools

- **ALMA** (Provided the low/intermediate-resolution large disc samples of 12CO integrated fluxes and CO-disc radii used as the observational input)
- **COpops** (Repository implementing the semi-analytical 12CO flux model used to generate model tracks for comparison with Upper Sco and Lupus data) — https://github.com/fzagaria/COpops
- **DALI** (Thermochemical models against which the semi-analytical flux model was benchmarked, recovering agreement within a factor of three)

## Examples

```
git clone https://github.com/fzagaria/COpops; python copops_model.py --region UpperSco --co-depletion --output upper_sco_model_comparison.csv
```

## Evaluation signals

- Flux-based usable sample exceeds size-based sample by the reported factors: ~3.6 for Upper Sco and ~1.3 for Lupus
- Model tracks agree with Upper Sco observations within the factor-of-three tolerance only when CO depletion is switched on
- Semi-analytical fluxes agree with DALI thermochemical models within a factor of three
- Output CSV contains per-disc and sample-level entries; PNG shows observed flux-size plane with overlaid model tracks

## Limitations

- Uncertainties on initial conditions limited interpretation of the observations
- Available ALMA data have not given conclusive answers on the underlying disc evolution mechanisms (viscosity vs MHD winds)
- The semi-analytical model relies on the assumption that 12CO is optically thick and is accurate only within a factor of three relative to thermochemical models

## Evidence

- [abstract] Sample sizes increase by 1.3 (Lupus) and 3.6 (Upper Sco) when studying fluxes instead of sizes: "increased samples, by a factor 1.3 (Lupus) and 3.6 (Upper Sco), when studying fluxes instead of sizes"
- [abstract] Models and data agree only when CO depletion is considered: "Models and data agree well only if CO depletion is considered"
- [abstract] Semi-analytical model benchmarked against DALI within factor of three: "benchmarked it against the results of DALI thermochemical models, recovering an agreement within a factor of three"
- [abstract] Optically thick assumption makes fluxes scale with disc surface area: "assuming that 12CO is optically thick, fluxes scale with the disc surface area"
- [readme] COpops README describes the method as fast, flexible, accurate within factor of three: "This method is fast, flexible and accurate: compared to more-complex physico-chemical models, we recover a good agreement within a factor of three."
