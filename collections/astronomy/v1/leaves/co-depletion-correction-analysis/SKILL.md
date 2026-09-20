---
name: co-depletion-correction-analysis
description: Use when comparing modelled 12CO line fluxes with ALMA disc-survey flux/size
  samples and you must test whether CO depletion from initial conditions reconciles
  models with observations.
when_to_use_negative:
- When the question is disc size (887-micron radius) evolution specifically and the
  sample is already flux-complete — the flux-based approach mainly helps because samples
  increase by factor 1.3 (Lupus) when using fluxes instead of sizes.
- When computationally-expensive thermochemical modelling per disc is required and
  budgeted — this skill exists as a fast, semi-analytical alternative validated against
  DALI within a factor of three, not a replacement for detailed chemistry.
- When the available ALMA data are expected to discriminate viscosity vs MHD winds
  conclusively — the article notes the data have not yet given conclusive answers
  on the underlying disc evolution mechanism.
edam_operation: http://edamontology.org/operation_0242
tools:
- name: ALMA
  role: Source of the 12CO line-flux and 887-micron disc-size survey observations
    (Lupus, Upper Sco) that the semi-analytical model fluxes are compared against
- name: DALI
  role: Thermochemical model used to benchmark the semi-analytical 12CO flux computation
    (agreement within a factor of three)
- name: COpops
  role: Repository implementing the semi-analytical disc CO flux modelling used for
    the population synthesis and model-data comparison
  repo: github.com/fzagaria/COpops
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202346164
    title: Testing protoplanetary disc evolution with CO fluxes
  source_content_hash: sha256:5218e8f5dfdbc35548389adf151a55063d61ecdc7cab3e24c12d7f87eaf42864
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALMA
  - COpops
  - DALI
  edam_operation: http://edamontology.org/operation_0242
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# co-depletion-correction-analysis

## Summary

A method for comparing semi-analytical 12CO flux models (viscous and MHD-wind tracks) against ALMA disc survey data (Lupus, Upper Sco) by bracketing CO-depletion uncertainty, which is the condition under which models and data agree.

## When to use

Use when comparing modelled 12CO line fluxes with ALMA disc-survey flux/size samples and you must test whether CO depletion from initial conditions reconciles models with observations.

## When NOT to use

- When the question is disc size (887-micron radius) evolution specifically and the sample is already flux-complete — the flux-based approach mainly helps because samples increase by factor 1.3 (Lupus) when using fluxes instead of sizes.
- When computationally-expensive thermochemical modelling per disc is required and budgeted — this skill exists as a fast, semi-analytical alternative validated against DALI within a factor of three, not a replacement for detailed chemistry.
- When the available ALMA data are expected to discriminate viscosity vs MHD winds conclusively — the article notes the data have not yet given conclusive answers on the underlying disc evolution mechanism.

## Inputs

- Lupus survey catalog of 12CO line fluxes and 887-micron disc sizes observed with ALMA
- Upper Sco disc sample (12CO fluxes and sizes)
- Semi-analytical 12CO flux model tracks (viscous and MHD-wind scenarios), run with and without CO depletion

## Outputs

- Overlay figure of model 12CO-flux/size tracks on observed Lupus (and Upper Sco) fluxes and sizes
- Summary table of model-vs-data agreement under CO-depletion vs non-depletion assumptions
- Sample-count comparison showing the factor 1.3 (Lupus) and 3.6 (Upper Sco) increase when using fluxes instead of sizes

## How to apply

Load the ALMA survey catalog of 12CO line fluxes and 887-micron disc sizes (e.g. the Lupus sample). Compute semi-analytical model 12CO fluxes for both viscous and MHD-wind evolutionary tracks, running the model twice — with CO depletion included and without — to bracket the initial-condition uncertainty. Overlay the model tracks on the observed fluxes and sizes: agreement is expected only in the CO-depletion case, as benchmarked by DALI agreement within a factor of three. Separately, count detectable discs twice, once with the 12CO flux criterion and once with the disc-size criterion, to quantify the usable sample increase (factor 1.3 for Lupus, 3.6 for Upper Sco) when studying fluxes instead of sizes. Assemble the model-vs-data comparison and sample-count comparison into a summary table and companion figure.

## Related tools

- **ALMA** (Source of the 12CO line-flux and 887-micron disc-size survey observations (Lupus, Upper Sco) that the semi-analytical model fluxes are compared against)
- **DALI** (Thermochemical model used to benchmark the semi-analytical 12CO flux computation (agreement within a factor of three))
- **COpops** (Repository implementing the semi-analytical disc CO flux modelling used for the population synthesis and model-data comparison) — github.com/fzagaria/COpops

## Evaluation signals

- Model tracks overlayed on Lupus data agree well only in the CO-depletion case; the non-depletion tracks should visibly miss the observed flux/size distribution.
- Semi-analytical fluxes agree with DALI thermochemical models within a factor of three.
- Recounting detectable Lupus discs with the 12CO flux criterion vs the size criterion reproduces the factor 1.3 sample increase (3.6 for Upper Sco).
- Both viscous and MHD-wind tracks are evaluated; conclusions should reflect that initial-condition uncertainties limited interpretation of the observations.
- The comparison output includes both a summary table and a companion figure for the Lupus condition.

## Limitations

- Uncertainties on the initial conditions limited interpretation of the observations — CO-depletion bracketing gives agreement but not a unique evolutionary solution.
- The available ALMA data have not given conclusive answers on the underlying disc evolution mechanisms (viscosity vs MHD winds).
- The semi-analytical method assumes optically thick 12CO, with fluxes scaling with the disc surface area; it is validated only to within a factor of three against DALI.
- The sample-size benefit of flux-based selection differs strongly by region: factor 1.3 for Lupus vs 3.6 for Upper Sco.

## Evidence

- [abstract] Models and data agree only if CO depletion is considered: "Models and data agree well only if CO depletion is considered"
- [abstract] Sample increase factor: "increased samples, by a factor 1.3 (Lupus) and 3.6 (Upper Sco), when studying fluxes instead of sizes"
- [abstract] DALI benchmark tolerance: "benchmarked it against the results of DALI thermochemical models, recovering an agreement within a factor of three"
- [readme] Method purpose: "Here we introduce an alternative method based on modelling semi-analytically the disc CO fluxes. This method is fast, flexible and accurate"
- [abstract] Open question on disc evolution mechanisms: "the available data have not given conclusive answers yet on the underlying disc evolution mechanisms: viscosity or magnetohydrodynamic (MHD) winds"
