---
name: radial-velocity-keplerian-orbit-fitting
description: Use when a transiting planet candidate has a radial-velocity time series
  (e.g. HARPS measurements) plus transit photometry, and you need masses, eccentricity,
  and credible intervals via joint Bayesian Keplerian fitting with model selection
  by evidence.
when_to_use_negative:
- The candidate is RV-only with no transit photometry — the joint fit as described
  requires simultaneous light curves to constrain the period and phase of the Keplerian
  signal
- The TESS aperture is heavily contaminated by nearby stars and no dilution correction
  is planned — TOI-5005 contributes only 84.4% of the Sector 65 photometry, so uncorrected
  contaminant flux biases transit depths and hence fitted radii
- The goal is exoplanet discovery/vetting rather than parameter inference — flux-contamination
  triage tools (e.g. TESS-cont DILUTION analysis) are the wrong model here
edam_operation: http://edamontology.org/operation_2945
tools:
- name: HARPS
  role: Source of the 38 radial velocity measurements fitted by the joint Bayesian
    Keplerian model
- name: TESS
  role: Three sectors of SPOC photometry providing transit depth and ephemeris constraints
    in the joint fit
- name: PEST
  role: Two ground-based transit light curves constraining the transit shape jointly
    with the RVs
- name: TRAPPIST-South
  role: Two additional ground-based transit light curves in the joint Bayesian model
- name: Python
  role: Environment for the Bayesian sampling and evidence computation scripts
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202451656
    title: 'TOI-5005 b: A super-Neptune in the savanna near the ridge'
  source_content_hash: sha256:7e58dbd519a55cf37e21090a9c0ecaff333623fa9952c0ae4a8ea8d49d35fdbd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HARPS
  - PEST
  - Python
  - TESS
  - TRAPPIST-South
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# radial-velocity-keplerian-orbit-fitting

## Summary

Joint Bayesian inference of a Keplerian signal against radial velocities, TESS photometry, and ground-based transit light curves to recover orbital and physical planet parameters (period, semi-amplitude, eccentricity, radius, mass), as done for the super-Neptune TOI-5005 b from 38 HARPS RVs and three TESS sectors.

## When to use

Use when a transiting planet candidate has a radial-velocity time series (e.g. HARPS measurements) plus transit photometry, and you need masses, eccentricity, and credible intervals via joint Bayesian Keplerian fitting with model selection by evidence.

## When NOT to use

- The candidate is RV-only with no transit photometry — the joint fit as described requires simultaneous light curves to constrain the period and phase of the Keplerian signal
- The TESS aperture is heavily contaminated by nearby stars and no dilution correction is planned — TOI-5005 contributes only 84.4% of the Sector 65 photometry, so uncorrected contaminant flux biases transit depths and hence fitted radii
- The goal is exoplanet discovery/vetting rather than parameter inference — flux-contamination triage tools (e.g. TESS-cont DILUTION analysis) are the wrong model here

## Inputs

- Radial velocity time series (e.g. 38 HARPS measurements, ESO archive)
- TESS SPOC PDCSAP photometry (three sectors, from MAST/ExoFOP)
- Ground-based transit light curves (two PEST and two TRAPPIST-South, from MAST/ExoFOP)
- Stellar parameters for converting semi-amplitude to planetary mass

## Outputs

- Posterior samples for orbital parameters (period, semi-amplitude, eccentricity, drift/GP hyperparameters)
- Physical parameter table with credible intervals (Rp, Mp, mean density, e.g. toi-5005b_orbital_parameters.csv)
- Bayesian evidence values per model variant (eccentric/circular, drift, GP)

## How to apply

Retrieve the RV time series (e.g. 38 HARPS measurements from the ESO archive) and the transit photometry (TESS SPOC light curves from MAST/ExoFOP, plus ground-based PEST and TRAPPIST-South light curves). Set up a joint Bayesian model in which the Keplerian signal is fitted simultaneously to the RVs and all transit light curves, and construct variant models covering eccentric vs circular orbits, an additional long-term drift term, and Gaussian-process activity terms to absorb stellar activity in the RVs. Compute the Bayesian evidence for each variant and select the model that best represents the combined dataset — this is the key decision point, since over-parameterized or under-parameterized models bias the semi-amplitude and hence the mass. Sample the posterior of the selected model and extract median values with credible intervals for period, semi-amplitude, eccentricity, Rp, and Mp, saving the parameter table (e.g. toi-5005b_orbital_parameters.csv). Validate by checking the recovered mass and radius against published/expected values; in the TOI-5005 case the fit must reproduce Rp = 6.25 ± 0.24 R⊕ and Mp = 32.7 ± 5.9 M⊕ within 1σ.

## Related tools

- **HARPS** (Source of the 38 radial velocity measurements fitted by the joint Bayesian Keplerian model)
- **TESS** (Three sectors of SPOC photometry providing transit depth and ephemeris constraints in the joint fit)
- **PEST** (Two ground-based transit light curves constraining the transit shape jointly with the RVs)
- **TRAPPIST-South** (Two additional ground-based transit light curves in the joint Bayesian model)
- **Python** (Environment for the Bayesian sampling and evidence computation scripts)

## Evaluation signals

- Recovered radius and mass agree with published values Rp = 6.25 ± 0.24 R⊕ and Mp = 32.7 ± 5.9 M⊕ within their 1σ credible intervals
- The selected model has the highest Bayesian evidence among the tested eccentric/circular, drift, and GP variants
- Derived mean density falls near the published 0.74 ± 0.16 g cm−3 (savanna-like low density, below ~1 g cm−3 ridge-savanna dividing line)
- Median parameter table with credible intervals is saved in the expected CSV schema (period, semi-amplitude, eccentricity, Rp, Mp)
- Gaussian-process activity terms, if selected, leave no residual periodic signal in the RV residuals at the stellar activity period

## Limitations

- Stellar activity can mimic or distort Keplerian signals, hence the need for GP activity terms and a long-term drift term as explicit model variants; omitting them risks biased masses
- TESS photometry is diluted by nearby sources — 15.6% of the Sector 65 flux comes from contaminants, and the second most contaminant star lies outside the TPF — so radii depend on the crowding correction used
- Model selection depends on the variant set tested; different GP kernels or drift choices can change which model wins the evidence comparison

## Evidence

- [abstract] Model variants and evidence-based selection: [evidence span withheld — anchor status: not_found]
- [abstract] Joint Bayesian dataset composition: "We used Bayesian inference to analyse 38 HARPS radial velocity measurements, three sectors of TESS photometry, and two PEST and TRAPPIST-South transits"
- [other] Validation criterion for the recovered parameters: [evidence span withheld — anchor status: not_found]
- [abstract] Photometric dilution caveat: "TOI-5005 contributes 84.4% to the measured photometry, while the remaining 15.6% comes from different nearby sources"
