---
name: gaussian-process-stellar-activity-modelling
description: Use when 38-point-class HARPS RV time series show correlated stellar
  activity noise that a plain Keplerian fit cannot absorb, and joint modelling with
  TESS sectors plus ground-based transits (PEST, TRAPPIST-South) is needed to separate
  planetary signal from activity jitter.
when_to_use_negative:
- When the suspected signal origin is dilution by nearby contaminant stars rather
  than stellar activity — use the TESS-cont DILUTION analysis instead.
- When the target has no RV measurements and only transit photometry is available,
  so no mass/amplitude signal exists for the GP to disentangle.
- When a simple Keplerian plus white-noise model already fits the RVs with no correlated
  residuals, adding a GP over-parameterises the fit and degrades mass precision.
tools:
- name: HARPS
  role: Provides the 38 radial velocity measurements in which the GP activity term
    models correlated stellar jitter
- name: TESS
  role: Supplies three sectors of SPOC photometry jointly fitted with the RVs so transit
    shape constrains the GP-deconvolved Keplerian
- name: PEST
  role: Ground-based transit light curves joined to the joint Bayesian fit to refine
    transit timing/depth
- name: TRAPPIST-South
  role: Ground-based transit light curves joined to the joint Bayesian fit to refine
    transit timing/depth
- name: Python
  role: Language in which the joint Bayesian/GP model, evidence computation, and posterior
    sampling are implemented
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202451656
    title: 'TOI-5005 b: A super-Neptune in the savanna near the ridge'
  source_content_hash: sha256:a727fdd9886ed23a15285c6b1c33a246d33c65f89bf9b16c3a1e885a1bc43dd4
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
yes_rate: 0.8571428571428571
n_claims: 7
---

# gaussian-process-stellar-activity-modelling

## Summary

A joint Bayesian inference recipe in which Gaussian-process activity terms are fitted simultaneously to HARPS radial velocities and TESS/ground-based transit photometry, with eccentric/circular, long-term-drift, and GP model variants discriminated via Bayesian evidence, to recover an unbiased planet mass and radius (as done for the super-Neptune TOI-5005 b).

## When to use

Use when 38-point-class HARPS RV time series show correlated stellar activity noise that a plain Keplerian fit cannot absorb, and joint modelling with TESS sectors plus ground-based transits (PEST, TRAPPIST-South) is needed to separate planetary signal from activity jitter.

## When NOT to use

- When the suspected signal origin is dilution by nearby contaminant stars rather than stellar activity — use the TESS-cont DILUTION analysis instead.
- When the target has no RV measurements and only transit photometry is available, so no mass/amplitude signal exists for the GP to disentangle.
- When a simple Keplerian plus white-noise model already fits the RVs with no correlated residuals, adding a GP over-parameterises the fit and degrades mass precision.

## Inputs

- 38 HARPS radial velocity measurements (from the ESO archive)
- Three sectors of TESS SPOC photometry (MAST/ExoFOP)
- Two PEST and two TRAPPIST-South ground-based transit light curves (MAST/ExoFOP)

## Outputs

- Posterior samples of the selected joint model (Keplerian + chosen activity/noise variant)
- Median values with credible intervals for period, semi-amplitude, eccentricity, Rp, Mp (e.g. Rp = 6.25 ± 0.24 R⊕, Mp = 32.7 ± 5.9 M⊕ for TOI-5005 b)
- Bayesian evidence values for each eccentric/circular, drift, and GP model variant
- Parameter table with credible intervals (toi-5005b_orbital_parameters.csv)

## How to apply

Retrieve the HARPS radial velocities from the ESO archive and the TESS SPOC PDCSAP photometry plus PEST and TRAPPIST-South transit light curves from MAST/ExoFOP. Build a joint Bayesian model that fits the Keplerian signal simultaneously to all datasets, and define variant models for eccentric vs circular orbits, an additional long-term drift term, and Gaussian-process activity terms to absorb correlated stellar noise. Compute the Bayesian evidence for every variant and select the model that best represents the combined dataset rather than assuming a GP a priori — a drift or circular model may suffice for a quiet star. Sample the posterior of the selected model and extract median values and credible intervals for period, semi-amplitude, eccentricity, Rp, and Mp, saving them with uncertainties (e.g. toi-5005b_orbital_parameters.csv). The GP must capture activity correlated noise without swallowing the Keplerian semi-amplitude, since the planet mass (Mp = 32.7 ± 5.9 M⊕ for TOI-5005 b) is the parameter most sensitive to this trade-off.

## Related tools

- **HARPS** (Provides the 38 radial velocity measurements in which the GP activity term models correlated stellar jitter)
- **TESS** (Supplies three sectors of SPOC photometry jointly fitted with the RVs so transit shape constrains the GP-deconvolved Keplerian)
- **PEST** (Ground-based transit light curves joined to the joint Bayesian fit to refine transit timing/depth)
- **TRAPPIST-South** (Ground-based transit light curves joined to the joint Bayesian fit to refine transit timing/depth)
- **Python** (Language in which the joint Bayesian/GP model, evidence computation, and posterior sampling are implemented)

## Evaluation signals

- The selected model variant should win on Bayesian evidence over the eccentric/circular, drift-only, and no-GP alternatives; report all evidence values.
- Recovered radius and mass must be consistent with the published values Rp = 6.25 ± 0.24 R⊕ and Mp = 32.7 ± 5.9 M⊕ within their 1σ credible intervals.
- Post-removal RV residuals should show no remaining autocorrelation structure at the GP-identified activity timescale.
- The inferred semi-amplitude must remain consistent across the GP and non-GP variants; a GP that absorbs the Keplerian signal will drive Mp toward zero.
- Derived mean density (0.74 ± 0.16 g cm−3 for TOI-5005 b) should be consistent with the independently constrained Rp and Mp posteriors.

## Limitations

- GP, drift, and orbital-eccentricity choices are confounded; model selection must rely on computed Bayesian evidence, and a mis-specified activity model biases the inferred mass.
- Ground-based photometry with limited coverage cannot by itself discard contaminant sources or constrain activity — higher-spatial resolution observations may still be needed to attribute signals.
- Downstream interpretations (e.g. atmospheric mass-loss rates, 0.013 ± 0.008 vs 0.17 ± 0.12 M⊕ Gyr−1) are strongly model-dependent, so GP-fit parameter uncertainties propagate into derived physical quantities.

## Evidence

- [abstract] Joint RV+photometry+transit scope: "We used Bayesian inference to analyse 38 HARPS radial velocity measurements, three sectors of TESS photometry, and two PEST and TRAPPIST-South transits"
- [abstract] Model variants including GPs: "We tested a set of models involving eccentric and circular orbits, long-term drifts, and Gaussian processes"
- [abstract] Evidence-based model selection: "We computed the Bayesian evidence to find the model that best represents our dataset"
- [readme] Activity-signal attribution context: "determines whether any of these sources could be responsible for the **transit** and/or **activity** signals."
