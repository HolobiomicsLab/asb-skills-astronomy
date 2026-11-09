---
name: laplace-approximation-posterior-inference
description: Use when deriving RVs and their uncertainties from echelle spectra (e.g.
  HARPS S2D, ESPRESSO) by template matching and you need a statistically principled
  posterior for a common RV shift per spectrum, marginalizing over the continuum.
when_to_use_negative:
- When the goal is the classical template-matching combination (per-order RV shifts
  combined by weighted average) rather than a common-shift semi-Bayesian posterior
- When input spectra are not echelle spectral orders comparable to a stellar template
  (e.g. already-combined RV time series or CCF outputs), since the skill operates
  on order-level S2D data
- When a full posterior beyond the Gaussian Laplace approximation is required (S-BART's
  sampler provides an approximate, not exact, posterior)
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_0621
tools:
- name: SBART
  role: pip-installable Python package that implements the semi-Bayesian template
    matching with the Laplace sampler and produces the RV time series
  repo: https://github.com/iastro-pt/sBART
- name: S-BART
  role: the pipeline/methodology that applies the semi-Bayesian approach to HARPS
    and ESPRESSO data
  repo: https://github.com/iastro-pt/sBART
- name: Telfit
  role: optional telluric absorption fitting used alongside SBART; installed independently
    and may require pysynphot
  repo: https://github.com/kgullikson88/Telluric-Fitter
- name: HARPS-TERRA
  role: baseline template-matching pipeline used to validate the RV scatter of the
    Laplace-based S-BART estimates
- name: SERVAL
  role: baseline template-matching pipeline used to validate the RV scatter of the
    Laplace-based S-BART estimates
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1051/0004-6361/202142262
    title: A novel framework for semi-Bayesian radial velocities through template
      matching
  source_content_hash: sha256:c7c204a24477cc936862bbddee70594200724eb11d55c8e729c7230d6fa8d8fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iastro-pt/sBART
  tools:
  - HARPS-TERRA
  - Laplace_sampler
  - S-BART
  - SBART
  - SERVAL
  - Telfit
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# laplace-approximation-posterior-inference

## Summary

Estimate per-order radial-velocity posteriors in a semi-Bayesian template-matching framework (S-BART) by maximizing the marginalized posterior with a Laplace (Gaussian) approximation around the MAP, instead of the classical order-wise weighted-average combination.

## When to use

Use when deriving RVs and their uncertainties from echelle spectra (e.g. HARPS S2D, ESPRESSO) by template matching and you need a statistically principled posterior for a common RV shift per spectrum, marginalizing over the continuum.

## When NOT to use

- When the goal is the classical template-matching combination (per-order RV shifts combined by weighted average) rather than a common-shift semi-Bayesian posterior
- When input spectra are not echelle spectral orders comparable to a stellar template (e.g. already-combined RV time series or CCF outputs), since the skill operates on order-level S2D data
- When a full posterior beyond the Gaussian Laplace approximation is required (S-BART's sampler provides an approximate, not exact, posterior)

## Inputs

- HARPS S2D spectral observations (e.g. archival observations of Barnard's star)
- ESPRESSO GTO spectra
- Stellar template built from the available observations
- DRIFT / DRIFT_ERR instrumental drift columns
- SBART user keywords (frame rejection flags, NORMALIZATION_MODE, number of workers)

## Outputs

- Posterior probability distributions for the relative RV of each spectrum (Laplace-approximated)
- RV time series rv_time_series.txt with columns BJD, RVc [Km/s], RVc_ERR [Km/s], SA [km/s], DRIFT [m/s], DRIFT_ERR, filename, frameIDs
- Order-wise RV uncertainty diagnostic plot RV_orderwise_errors.png

## How to apply

Load the HARPS S2D observations (e.g. Barnard's star archival data) and the stellar template built from the available observations into the SBART DataClass, then launch the RV_routine with the Laplace sampler. Enforce a single common RV shift across all spectral orders of each spectrum while marginalizing over the continuum, so the posterior probability distribution for the relative RV is obtained via the Laplace approximation rather than by fitting each order independently. Contrast this with the classical template-matching variant, in which a RV shift is estimated individually for each spectral order and combined through a weighted average. Apply the instrumental drift correction (DRIFT column, added in quadrature to raw RVs when not nan) and the secular acceleration correction before writing the final RVc column in km/s. SBART is pip-installable (`pip install SBART`) and supports HARPS and ESPRESSO data; telluric handling can use Telfit and pysynphot, installed independently.

## Related tools

- **SBART** (pip-installable Python package that implements the semi-Bayesian template matching with the Laplace sampler and produces the RV time series) — https://github.com/iastro-pt/sBART
- **S-BART** (the pipeline/methodology that applies the semi-Bayesian approach to HARPS and ESPRESSO data) — https://github.com/iastro-pt/sBART
- **Telfit** (optional telluric absorption fitting used alongside SBART; installed independently and may require pysynphot) — https://github.com/kgullikson88/Telluric-Fitter
- **HARPS-TERRA** (baseline template-matching pipeline used to validate the RV scatter of the Laplace-based S-BART estimates)
- **SERVAL** (baseline template-matching pipeline used to validate the RV scatter of the Laplace-based S-BART estimates)

## Examples

```
pip install SBART  # then launch the SBART RV_routine with the Laplace sampler on HARPS S2D spectra and the stellar template
```

## Evaluation signals

- The standard deviation of the S-BART RVs on Barnard's star HARPS data is smaller than that obtained with HARPS-TERRA and SERVAL
- No single spectral order shows an anomalously small uncertainty in the RV_orderwise_errors.png diagnostic plot
- rv_time_series.txt contains all default columns (BJD, RVc [Km/s], RVc_ERR [Km/s], SA [km/s], DRIFT [m/s], DRIFT_ERR, filename, frameIDs) and RVc is in km/s
- DRIFT values that are not nan are added in quadrature to the raw RVs before writing RVc
- A single common RV shift is fitted across all orders of each spectrum, not one RV per order

## Limitations

- Sigma-clipping of RVs based on the DRS estimate, frame rejection based on warning KW flags, and MAD-based flux-outlier flagging must be configured by the user; the pipeline alone does not guarantee clean inputs
- Telfit/pysynphot must be installed independently and can fail (e.g. missing GDAS profiles for some reference frames, for which fallback templates exist)
- Attributed performance differences relative to other pipelines stem from template construction and telluric handling, so results depend on those upstream choices
- Performance was validated only against HARPS (Barnard's star) and ESPRESSO GTO data; other instruments are not covered by the article's validation

## Evidence

- [abstract] posterior RV via Laplace approximation with continuum marginalization: "a common RV shift is assumed to describe the difference between each spectral order of a given stellar spectrum and a template built from the available observations."
- [abstract] classical template matching contrast: "a RV shift is estimated individually for each spectral order and the final RV estimate is calculated as a weighted average of the RVs of the individual orders."
- [abstract] S-BART outperforms baselines: "the standard deviation of the RVs derived with S-BART is smaller than that obtained with the HARPS-TERRA and SERVAL pipelines"
- [methods] SBART installation: "SBART can be pip-installed:

    pip install SBART"
- [abstract] instrument applicability: "This methodology, named the Semi-Bayesian Approach for RVs with Template matching, S-BART, can currently be applied to HARPS and ESPRESSO data."
- [methods] optional telluric dependencies: "To use `Telfit <https://github.com/kgullikson88/Telluric-Fitter>`_. it must be installed independently:

    pip install Telfit"
