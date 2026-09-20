---
name: bayesian-evidence-model-comparison
description: Use when jointly fitting Keplerian signals to HARPS radial velocities
  and TESS plus ground-based transit photometry, and competing model variants (eccentric
  vs circular, drift, GP activity) must be compared by Bayesian evidence.
when_to_use_negative:
- When the goal is to quantify flux contamination from nearby stars in the TESS aperture
  (e.g. the 84.4% target contribution for TOI-5005 in Sector 65) — use TESS-cont instead.
- When only a single model can be defined over the data — Bayesian evidence comparison
  requires at least two competing variants.
- When radial velocities or transit photometry are unavailable, since the joint fit
  cannot constrain both mass and radius.
tools:
- name: Python
  role: Language used to set up the joint Bayesian model of RVs and photometry and
    to compute the Bayesian evidence for each model variant
- name: HARPS
  role: Provides the 38 radial velocity measurements jointly fitted in the Bayesian
    model
- name: TESS
  role: Provides the three sectors of SPOC photometry jointly fitted in the Bayesian
    model
- name: PEST
  role: Provides two ground-based transit light curves included in the joint fit
- name: TRAPPIST-South
  role: Provides two ground-based transit light curves included in the joint fit
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202451656
    title: 'TOI-5005 b: A super-Neptune in the savanna near the ridge'
  source_content_hash: sha256:9763f8c2f863aed766d946ed2cf2f923c9f5ee0c86eb66856507b68f44e4eec3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HARPS
  - PEST
  - Python
  - TESS
  - TRAPPIST-South
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# bayesian-evidence-model-comparison

## Summary

Joint Bayesian inference that fits a Keplerian model simultaneously to 38 HARPS radial velocities, three sectors of TESS photometry, and two PEST plus two TRAPPIST-South ground-based transits, selecting among eccentric/circular, long-term-drift, and Gaussian-process model variants via Bayesian evidence. Applied to TOI-5005 b, it recovers Rp = 6.25 ± 0.24 R⊕ and Mp = 32.7 ± 5.9 M⊕.

## When to use

Use when jointly fitting Keplerian signals to HARPS radial velocities and TESS plus ground-based transit photometry, and competing model variants (eccentric vs circular, drift, GP activity) must be compared by Bayesian evidence.

## When NOT to use

- When the goal is to quantify flux contamination from nearby stars in the TESS aperture (e.g. the 84.4% target contribution for TOI-5005 in Sector 65) — use TESS-cont instead.
- When only a single model can be defined over the data — Bayesian evidence comparison requires at least two competing variants.
- When radial velocities or transit photometry are unavailable, since the joint fit cannot constrain both mass and radius.

## Inputs

- 38 HARPS radial velocity measurements of TOI-5005 retrieved from the ESO archive
- Three sectors of TESS SPOC photometry retrieved from MAST/ExoFOP
- Two PEST and two TRAPPIST-South ground-based transit light curves retrieved from MAST/ExoFOP

## Outputs

- Selected model variant (eccentric vs circular orbit, long-term drift, Gaussian-process activity terms) chosen by Bayesian evidence
- Posterior median values and 1σ credible intervals for orbital and physical parameters (period, semi-amplitude, eccentricity, Rp, Mp)
- Parameter table saved as toi-5005b_orbital_parameters.csv with credible intervals

## How to apply

Retrieve the 38 HARPS radial velocities from the ESO archive, the three TESS SPOC sectors from MAST/ExoFOP, and the two PEST and two TRAPPIST-South transit light curves from MAST/ExoFOP, then build a joint Bayesian model fitting the Keplerian signal to all datasets at once. Define variant models covering eccentric versus circular orbits, an additional long-term drift term, and Gaussian-process activity terms. Compute the Bayesian evidence for each variant and select the model that best represents the combined dataset rather than judging by fit quality alone. Sample the posterior of the selected model and extract median values with 1σ credible intervals for period, semi-amplitude, eccentricity, radius Rp, and mass Mp. Save the inferred parameter table with credible intervals to toi-5005b_orbital_parameters.csv and validate that the recovered radius and mass are consistent with the published values Rp = 6.25 ± 0.24 R⊕ and Mp = 32.7 ± 5.9 M⊕ within their 1σ credible intervals.

## Related tools

- **Python** (Language used to set up the joint Bayesian model of RVs and photometry and to compute the Bayesian evidence for each model variant)
- **HARPS** (Provides the 38 radial velocity measurements jointly fitted in the Bayesian model)
- **TESS** (Provides the three sectors of SPOC photometry jointly fitted in the Bayesian model)
- **PEST** (Provides two ground-based transit light curves included in the joint fit)
- **TRAPPIST-South** (Provides two ground-based transit light curves included in the joint fit)

## Evaluation signals

- Recovered radius and mass must be consistent with the published values Rp = 6.25 ± 0.24 R⊕ and Mp = 32.7 ± 5.9 M⊕ within their 1σ credible intervals
- Bayesian evidence must be computed for every model variant tested (eccentric, circular, long-term drift, Gaussian processes), not an ad hoc model choice
- Median values should match the published joint fit: Rp = 0.558 ± 0.021 RJ and Mp = 0.103 ± 0.018 MJ, implying mean density consistent with 0.74 ± 0.16 g cm−3
- Output CSV must contain credible intervals for all orbital and physical parameters, not point estimates alone

## Limitations

- Model comparison is conditional on the tested set of variants (eccentric vs circular orbits, long-term drifts, Gaussian processes); other model families are not covered.
- Posterior credible intervals for Rp and Mp are only meaningful if the selected model adequately represents the combined dataset.
- Validation assumes the same data retrieval (38 HARPS RVs from ESO, three TESS sectors, PEST/TRAPPIST-South transits); different data cuts may fail the 1σ consistency check.

## Evidence

- [abstract] We used Bayesian inference to analyse 38 HARPS radial velocity measurements, three sectors of TESS photometry, and two PEST and TRAPPIST-South transits: "We used Bayesian inference to analyse 38 HARPS radial velocity measurements, three sectors of TESS photometry, and two PEST and TRAPPIST-South transits"
- [abstract] We tested a set of models involving eccentric and circular orbits, long-term drifts, and Gaussian processes: "We tested a set of models involving eccentric and circular orbits, long-term drifts, and Gaussian processes"
- [abstract] We computed the Bayesian evidence to find the model that best represents our dataset: "We computed the Bayesian evidence to find the model that best represents our dataset"
