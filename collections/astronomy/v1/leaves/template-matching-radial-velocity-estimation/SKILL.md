---
name: template-matching-radial-velocity-estimation
description: Use when extracting RV time series from echelle S2D spectra (HARPS or
  ESPRESSO) and you need a statistically principled per-spectrum RV posterior instead
  of a per-order weighted average of template-matching shifts.
when_to_use_negative:
- The spectra are not from HARPS or ESPRESSO — S-BART 'can currently be applied to
  HARPS and ESPRESSO data' only
- The goal is a full analysis of planetary or activity signals in the RV series —
  the paper explicitly leaves this for future work
- The spectra are already reduced to CCF-based RVs by the ESO official pipeline and
  only a cross-method scatter comparison is needed without re-extraction
tools:
- name: SBART / S-BART
  role: 'Main pipeline: DataClass loading, stellar/telluric template construction,
    and Laplace-sampler RV extraction with common per-spectrum RV shift'
  repo: https://github.com/iastro-pt/sBART
- name: Telfit
  role: Telluric absorption model used to build the telluric template from the same
    observations, driven by downloaded GDAS atmospheric profiles
  repo: https://github.com/kgullikson88/Telluric-Fitter
- name: HARPS-TERRA
  role: Reference template-matching pipeline used as the precision benchmark on Barnard's
    star HARPS data
- name: SERVAL
  role: Reference template-matching pipeline used as the precision benchmark on Barnard's
    star HARPS data
provenance:
  source_task_ids:
  - task_007
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202142262
    title: A novel framework for semi-Bayesian radial velocities through template
      matching
  source_content_hash: sha256:084d8f9ea23c6f17cc40b6b5d41f9c59810f3d767886b2ae209df83830a44b52
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iastro-pt/sBART
  tools:
  - HARPS-TERRA
  - S-BART
  - SBART
  - SBART / S-BART
  - SERVAL
  - Telfit
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# template-matching-radial-velocity-estimation

## Summary

Estimate precise stellar radial velocities and uncertainties by template matching stellar spectra against a template built from available observations, using S-BART's semi-Bayesian formulation with a Laplace-approximated posterior. It was validated on HARPS archival data of Barnard's star against HARPS-TERRA and SERVAL.

## When to use

Use when extracting RV time series from echelle S2D spectra (HARPS or ESPRESSO) and you need a statistically principled per-spectrum RV posterior instead of a per-order weighted average of template-matching shifts.

## When NOT to use

- The spectra are not from HARPS or ESPRESSO — S-BART 'can currently be applied to HARPS and ESPRESSO data' only
- The goal is a full analysis of planetary or activity signals in the RV series — the paper explicitly leaves this for future work
- The spectra are already reduced to CCF-based RVs by the ESO official pipeline and only a cross-method scatter comparison is needed without re-extraction

## Inputs

- HARPS or ESPRESSO S2D echelle spectra (e.g. HARPS archival observations of Barnard's star)
- Stellar template built from the available observations of the target
- Telluric template created from the same observations via Telfit with GDAS atmospheric profiles
- Instrumental drift measurements (DRIFT column values) for the frames

## Outputs

- RV time series file rv_time_series.txt with columns BJD, RVc [Km/s], RVc_ERR [Km/s], SA [km/s], DRIFT [m/s], DRIFT_ERR, filename, frameIDs
- Posterior probability distributions for the relative RV of each spectrum (Laplace approximation, continuum-marginalized)
- Diagnostic plots RV_orderwise_errors.png and DataRejectionSummary.png under RV_step/individual_subInst/<instrument>/

## How to apply

Load the S2D spectra (e.g. HARPS archival observations of Barnard's star) into the SBART DataClass, applying the frame filters: order-wise SNR check, rejection based on warning keyword flags, and MAD-based flux outlier flagging. Build the stellar template from the available observations and the telluric template from the same data (Telfit with downloaded GDAS atmospheric profiles). Launch the RV routine with the Laplace sampler, assuming a single common RV shift across all spectral orders of each spectrum while marginalizing over the continuum to obtain the posterior RV; for comparison, run the classical variant that estimates the RV individually per order and combines them through a weighted average. Correct the raw RVs for instrumental drift (DRIFT column, added in quadrature when not nan) and secular acceleration (SA, DACE time origin) to produce RVc in km/s, and inspect RV_orderwise_errors.png to reject pathological orders with anomalously small uncertainties.

## Related tools

- **SBART / S-BART** (Main pipeline: DataClass loading, stellar/telluric template construction, and Laplace-sampler RV extraction with common per-spectrum RV shift) — https://github.com/iastro-pt/sBART
- **Telfit** (Telluric absorption model used to build the telluric template from the same observations, driven by downloaded GDAS atmospheric profiles) — https://github.com/kgullikson88/Telluric-Fitter
- **HARPS-TERRA** (Reference template-matching pipeline used as the precision benchmark on Barnard's star HARPS data)
- **SERVAL** (Reference template-matching pipeline used as the precision benchmark on Barnard's star HARPS data)

## Examples

```
pip install SBART
```

## Evaluation signals

- The standard deviation of the S-BART RVs on Barnard's star HARPS data is smaller than that obtained with HARPS-TERRA and SERVAL
- RV_orderwise_errors.png shows no single order with an anomalously small RV uncertainty
- rv_time_series.txt exists with all default columns (BJD, RVc, RVc_ERR, SA, DRIFT, DRIFT_ERR, filename, frameIDs) populated
- A single common RV shift was enforced across all spectral orders per spectrum (not order-wise estimates combined by weighted average)
- DataRejectionSummary.png confirms frames were filtered by order-wise SNR, warning KW flags, and MAD flux-outlier flagging before template building

## Limitations

- A full analysis of the planetary and activity signals present in the considered datasets was left for future work
- Performance differences versus other pipelines are attributed to template construction and telluric handling, so telluric-template quality (Telfit + GDAS profiles) directly affects results
- Telfit requires pysynphot and independent installation, and there is a known fallback path when GDAS profiles cannot be downloaded for some reference frames

## Evidence

- [abstract] common RV shift per spectrum with posterior RVs obtained via Laplace approximation after marginalization over the continuum: "a common RV shift is assumed to describe the difference between each spectral order of a given stellar spectrum and a template built from the available observations."
- [abstract] classical template-matching comparison baseline: "a RV shift is estimated individually for each spectral order and the final RV estimate is calculated as a weighted average of the RVs of the individual orders."
- [abstract] precision benchmark against HARPS-TERRA and SERVAL: "the standard deviation of the RVs derived with S-BART is smaller than that obtained with the HARPS-TERRA and SERVAL pipelines"
- [methods] frame filtering before template construction: "Added a new condition to check order-wise SNR"
- [readme] S-BART identity and scope: "s-BART: A semi-Bayesian implementation of template matching for precise Radial Velocities"
