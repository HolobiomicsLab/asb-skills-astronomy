---
name: per-order-rv-weighted-average-combination
description: Use when deriving an RV time series from echelle spectra (e.g., HARPS
  S2D) with the classical template-matching formulation, or when establishing a per-order
  baseline to diagnose whether a semi-Bayesian approach is needed.
when_to_use_negative:
- When the goal is the statistically principled semi-Bayesian posterior — S-BART marginalizes
  over the continuum and enforces a single common RV shift across all orders, and
  on Barnard's star it yields smaller RV scatter than the per-order weighted-average
  pipelines (HARPS-TERRA, SERVAL)
- When an order shows an anomalously small uncertainty in RV_orderwise_errors.png
  — the weighted average can then be dominated by an unreliable order
edam_operation: http://edamontology.org/operation_3234
tools:
- name: S-BART
  role: Pipeline in which the classical per-order template matching variant is implemented
    as the validation baseline; it also writes the order-wise uncertainty diagnostic
  repo: https://github.com/iastro-pt/sBART
- name: HARPS-TERRA
  role: External template-matching pipeline using the per-order weighted-average scheme,
    used for comparison of RV standard deviation
- name: SERVAL
  role: External template-matching pipeline using the per-order weighted-average scheme,
    used for comparison of RV standard deviation
- name: HARPS
  role: Instrument providing the S2D spectral orders to which the per-order RV estimation
    is applied
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1051/0004-6361/202142262
    title: A novel framework for semi-Bayesian radial velocities through template
      matching
  source_content_hash: sha256:f4b8defb588757a020e6b15519538facf4bbe53a3cd9e4280c6b4d1718bde7f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iastro-pt/sBART
  tools:
  - HARPS
  - HARPS-TERRA
  - S-BART
  - SBART
  - SERVAL
  edam_operation: http://edamontology.org/operation_3234
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# per-order-rv-weighted-average-combination

## Summary

Classical template-matching baseline for stellar radial velocities: estimate an RV shift independently for each spectral order against the stellar template, then combine the order-wise RVs into a single spectrum-level value through a weighted average. Serves as the reference point against which the semi-Bayesian (S-BART) single-common-RV-shift formulation is validated.

## When to use

Use when deriving an RV time series from echelle spectra (e.g., HARPS S2D) with the classical template-matching formulation, or when establishing a per-order baseline to diagnose whether a semi-Bayesian approach is needed.

## When NOT to use

- When the goal is the statistically principled semi-Bayesian posterior — S-BART marginalizes over the continuum and enforces a single common RV shift across all orders, and on Barnard's star it yields smaller RV scatter than the per-order weighted-average pipelines (HARPS-TERRA, SERVAL)
- When an order shows an anomalously small uncertainty in RV_orderwise_errors.png — the weighted average can then be dominated by an unreliable order

## Inputs

- HARPS S2D echelle spectral orders of a target star (e.g., Barnard's star archival observations)
- Stellar template built from the available observations
- DRIFT and DRIFT_ERR instrumental drift columns
- Order-wise RV uncertainty estimates

## Outputs

- Combined per-spectrum RV (RVc [Km/s]) with uncertainty (RVc_ERR [Km/s])
- Order-wise RV values and uncertainties (diagnostic plot RV_orderwise_errors.png)
- RV time series file (rv_time_series.txt) with columns BJD, RVc, RVc_ERR, SA [km/s], DRIFT [m/s], DRIFT_ERR, filename, frameIDs

## How to apply

Build a stellar template from the available observations (e.g., HARPS archival S2D frames of Barnard's star), then fit the RV shift between each spectral order of a given spectrum and that template individually, rather than enforcing one common shift. Combine the order-wise RVs through a weighted average of the individual order RVs to get the final per-spectrum value. Keep the order-wise uncertainties as diagnostics: if a single order shows an anomalously small uncertainty, the weighting is unreliable. Because this per-order treatment discards the correlation structure between orders, compare its RV scatter against the semi-Bayesian Laplace-approximation result — on Barnard's star HARPS data, S-BART achieved smaller standard deviation than this classical scheme as implemented in HARPS-TERRA and SERVAL. Correct the combined RVs with the instrumental drift (DRIFT column, added in quadrature when not NaN) and secular acceleration before forming the RVc time series.

## Related tools

- **S-BART** (Pipeline in which the classical per-order template matching variant is implemented as the validation baseline; it also writes the order-wise uncertainty diagnostic) — https://github.com/iastro-pt/sBART
- **HARPS-TERRA** (External template-matching pipeline using the per-order weighted-average scheme, used for comparison of RV standard deviation)
- **SERVAL** (External template-matching pipeline using the per-order weighted-average scheme, used for comparison of RV standard deviation)
- **HARPS** (Instrument providing the S2D spectral orders to which the per-order RV estimation is applied)

## Examples

```
pip install SBART
```

## Evaluation signals

- Order-wise RV uncertainties are finite and no single order shows an anomalously small uncertainty in RV_orderwise_errors.png
- The combined RVc values carry the instrumental DRIFT correction added in quadrature (when not NaN) and the secular acceleration (SA) correction
- rv_time_series.txt contains the expected columns: BJD, RVc [Km/s], RVc_ERR [Km/s], SA [km/s], DRIFT [m/s], DRIFT_ERR, filename, frameIDs
- The standard deviation of the combined RVs is compared against HARPS-TERRA and SERVAL; the classical per-order scheme is expected to show equal or larger scatter than S-BART on Barnard's star

## Limitations

- This per-order scheme does not marginalize over the continuum and does not assume a common RV shift, so it is less statistically principled than the semi-Bayesian Laplace approximation
- Performance differences relative to other pipelines were attributed by the authors to differences in stellar template construction and telluric handling
- Instrumental drift corrections may be NaN for some frames; they are only added in quadrature when not NaN

## Evidence

- [abstract] per-order weighted average baseline: "a RV shift is estimated individually for each spectral order and the final RV estimate is calculated as a weighted average of the RVs of the individual orders."
- [abstract] S-BART outperforms classical pipelines on scatter: "the standard deviation of the RVs derived with S-BART is smaller than that obtained with the HARPS-TERRA and SERVAL pipelines"
- [readme] semi-Bayesian alternative formulation: "s-BART: A semi-Bayesian implementation of template matching for precise Radial Velocities"
- [methods] installable tooling for the pipeline: "SBART can be pip-installed:

    pip install SBART"
