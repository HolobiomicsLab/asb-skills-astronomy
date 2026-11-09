---
name: joint-photometric-rv-bayesian-inference
description: Use when combining radial velocities and transit light curves (space
  + ground) to derive planet mass, radius, and orbital parameters with Bayesian evidence-based
  model comparison and posterior credible intervals.
when_to_use_negative:
- Only one data type is available (e.g. photometry without RVs) — the mass and density
  cannot be constrained, so the joint fit is not applicable
- The TESS aperture contamination has not been characterised — run TESS-cont first,
  since nearby sources contributed 15.6% of the Sector 65 flux and could bias dilution-corrected
  depths
- The goal is composition/atmospheric-loss modelling rather than orbital/physical
  parameter inference (e.g. internal-structure modelling or photoevaporation rates
  are separate downstream steps)
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: HARPS
  role: Provides the 38 radial velocity measurements constraining the Keplerian semi-amplitude
    and eccentricity
- name: TESS
  role: Three sectors of SPOC photometry fixing the transit shape, period, and planetary
    radius
- name: PEST
  role: Two ground-based transit light curves included in the joint fit
- name: TRAPPIST-South
  role: Two ground-based transit light curves included in the joint fit
- name: TESS-cont
  role: Pre-fit check of flux contamination/dilution in the TESS aperture (CROWDSAP/FLFRCSAP
    by-products) before photometry enters the joint model
  repo: https://github.com/castro-gzlz/TESS-cont
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202451656
    title: 'TOI-5005 b: A super-Neptune in the savanna near the ridge'
  source_content_hash: sha256:aa8c7cbafce4a347b910de3374c62d03730dd9881cf42d35fa1dd3f9ce275727
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/castro-gzlz/TESS-cont
  tools:
  - HARPS
  - PEST
  - Python
  - TESS
  - TESS-cont
  - TRAPPIST-South
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# joint-photometric-rv-bayesian-inference

## Summary

Joint Bayesian inference of a planet's mass, radius, and orbit by fitting a Keplerian + transit model simultaneously to radial velocities (e.g. 38 HARPS measurements), multi-sector TESS photometry, and ground-based transit light curves, with model selection by Bayesian evidence.

## When to use

Use when combining radial velocities and transit light curves (space + ground) to derive planet mass, radius, and orbital parameters with Bayesian evidence-based model comparison and posterior credible intervals.

## When NOT to use

- Only one data type is available (e.g. photometry without RVs) — the mass and density cannot be constrained, so the joint fit is not applicable
- The TESS aperture contamination has not been characterised — run TESS-cont first, since nearby sources contributed 15.6% of the Sector 65 flux and could bias dilution-corrected depths
- The goal is composition/atmospheric-loss modelling rather than orbital/physical parameter inference (e.g. internal-structure modelling or photoevaporation rates are separate downstream steps)

## Inputs

- 38 HARPS radial velocity measurements (ESO archive)
- Three sectors of TESS SPOC photometry (MAST/ExoFOP)
- Two PEST ground-based transit light curves (MAST/ExoFOP)
- Two TRAPPIST-South ground-based transit light curves (MAST/ExoFOP)
- Stellar parameters for converting RV semi-amplitude and transit depth to planetary mass and radius

## Outputs

- Posterior median orbital parameters with 1σ credible intervals (period, semi-amplitude, eccentricity, radius Rp, mass Mp)
- Bayesian evidence values for each model variant (eccentric/circular, drift, GP)
- Parameter table saved as toi-5005b_orbital_parameters.csv
- Derived mean density (e.g. ρp = 0.74 ± 0.16 g cm−3 for TOI-5005 b)

## How to apply

Retrieve the HARPS radial velocities (38 measurements of TOI-5005 from the ESO archive), the TESS SPOC photometry (three sectors, from MAST/ExoFOP), and the ground-based transit light curves (two PEST and two TRAPPIST-South). Build a joint model fitting the Keplerian signal simultaneously to RVs and all photometry, then test model variants: eccentric vs circular orbit, an additional long-term drift term, and Gaussian-process activity terms, computing the Bayesian evidence for each variant to select the best representation of the dataset. Sample the posterior of the winning model and report median values with 1σ credible intervals for period, semi-amplitude, eccentricity, radius, and mass. Because SPOC photometry is dilution-corrected, first verify contamination is handled (TOI-5005 contributes 84.4% of Sector 65 flux, 15.6% from nearby sources) using DR2-based metrics, since the TIC/SPOC catalogue is stacked to Gaia DR2.

## Related tools

- **HARPS** (Provides the 38 radial velocity measurements constraining the Keplerian semi-amplitude and eccentricity)
- **TESS** (Three sectors of SPOC photometry fixing the transit shape, period, and planetary radius)
- **PEST** (Two ground-based transit light curves included in the joint fit)
- **TRAPPIST-South** (Two ground-based transit light curves included in the joint fit)
- **TESS-cont** (Pre-fit check of flux contamination/dilution in the TESS aperture (CROWDSAP/FLFRCSAP by-products) before photometry enters the joint model) — https://github.com/castro-gzlz/TESS-cont

## Evaluation signals

- Recovered radius matches Rp = 6.25 ± 0.24 R⊕ and mass matches Mp = 32.7 ± 5.9 M⊕ within 1σ credible intervals (the task's stated validation)
- Mean density of the posterior sample is consistent with 0.74 ± 0.16 g cm−3
- The selected model attains the highest Bayesian evidence among the tested eccentric/circular, drift, and Gaussian-process variants
- Exported CSV contains period, semi-amplitude, eccentricity, Rp, and Mp each with credible intervals
- Photometric dilution metrics agree with official TESS values (no major DR2/DR3 discrepancy flagged)

## Limitations

- Model choice matters: eccentric vs circular orbits, long-term drifts, and GP activity terms must be compared via Bayesian evidence rather than fixed a priori
- TESS photometry is dilution-corrected and the TIC/SPOC catalogue is stacked to Gaia DR2, so contamination metrics should be computed with the DR2 catalogue and cross-checked against DR3; discrepancies between DR2 and DR3 catalogues can shift CROWDSAP/FLFRCSAP
- TESS-cont's custom aperture generation feature is undocumented, limiting customisation of the photometric input

## Evidence

- [abstract] Joint fit scope: "We used Bayesian inference to analyse 38 HARPS radial velocity measurements, three sectors of TESS photometry, and two PEST and TRAPPIST-South transits"
- [abstract] Model variants tested: "We tested a set of models involving eccentric and circular orbits, long-term drifts, and Gaussian processes"
- [abstract] Model selection criterion: "We computed the Bayesian evidence to find the model that best represents our dataset"
- [abstract] Validation target values: "We confirm TOI-5005 b to be a transiting super-Neptune with a radius of R p = 6.25 ± 0.24 R ⊕"
- [readme] Dilution caveat for photometric input: "TOI-5005 contributes 84.4% to the measured photometry, while the remaining 15.6% comes from different nearby sources"
