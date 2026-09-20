---
name: rv-time-series-precision-comparison
description: Use when you have RV time series from multiple template-matching pipelines
  (S-BART, HARPS-TERRA, SERVAL) on the same spectrograph dataset and need to compare
  their scatter/precision quantitatively.
when_to_use_negative:
- The planetary or activity signals in the RV time series need to be analyzed, not
  just the scatter — the authors explicitly leave a full analysis of the planetary
  and activity signals for future work
- The goal is CCF-based (ESO DRS) RV comparison rather than template matching vs template
  matching; a separate comparison against the CCF method applies
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: S-BART
  role: Produces the S-BART RV time series from HARPS spectra whose standard deviation
    is benchmarked against other pipelines
  repo: https://github.com/iastro-pt/sBART
- name: Telfit
  role: Telluric absorption model used to build the telluric template from the same
    observations, affecting the RV precision being compared
  repo: https://github.com/kgullikson88/Telluric-Fitter
- name: HARPS-TERRA
  role: Reference template-matching pipeline whose RV standard deviation provides
    the baseline for comparison
- name: SERVAL
  role: Reference template-matching pipeline whose RV standard deviation provides
    the baseline for comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202142262
    title: A novel framework for semi-Bayesian radial velocities through template
      matching
  source_content_hash: sha256:205c486d06efcf86ecb17b8935de95eea8ce9bd80ae641c933b7a5f8b0313b5f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iastro-pt/sBART
  tools:
  - HARPS-TERRA
  - S-BART
  - SBART
  - SERVAL
  - Telfit
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# rv-time-series-precision-comparison

## Summary

Benchmark the RV precision (standard deviation) of S-BART template-matching RVs against other template matching pipelines (HARPS-TERRA, SERVAL) on the same HARPS archival time series, e.g. Barnard's star.

## When to use

Use when you have RV time series from multiple template-matching pipelines (S-BART, HARPS-TERRA, SERVAL) on the same spectrograph dataset and need to compare their scatter/precision quantitatively.

## When NOT to use

- The planetary or activity signals in the RV time series need to be analyzed, not just the scatter — the authors explicitly leave a full analysis of the planetary and activity signals for future work
- The goal is CCF-based (ESO DRS) RV comparison rather than template matching vs template matching; a separate comparison against the CCF method applies

## Inputs

- HARPS archival spectral observations (e.g. of Barnard's star)
- S-BART RV time-series text file with default columns: filename, RV (km/s, drift- and SA-corrected), RV_ERR, DRS_RV, BERV, SA, DRIFT, DRIFT_ERR
- RV time series from comparison pipelines (HARPS-TERRA, SERVAL) on the same observations
- GDAS atmospheric profiles for Telfit telluric modeling

## Outputs

- Standard deviation of the S-BART RV time series
- Comparison of S-BART RV scatter vs HARPS-TERRA and SERVAL RV scatter
- Per-order uncertainty diagnostics (RV_orderwise_errors.png) and DataRejection summary (DataRejection.png) under RV_step/individual/<instrument>/

## How to apply

Run S-BART on HARPS archival observations (e.g. of Barnard's star), loading frames into the SBART DataClass with the order-wise SNR check, rejection on warning keyword flags, and MAD-based flux outlier flagging. Build the stellar template and the telluric template from the same observations using Telfit driven by GDAS atmospheric profiles, then launch RV extraction where a common RV shift is fitted per spectrum with posterior RVs obtained via Laplace approximation after marginalizing over the continuum. Apply the instrumental drift (DRIFT column) and secular acceleration (SA, DACE time origin) corrections to the extracted RVs. Finally compute the standard deviation of the resulting S-BART RV time series and compare it against the standard deviations of the HARPS-TERRA and SERVAL RVs for the same data; differences are attributed to template construction and telluric handling.

## Related tools

- **S-BART** (Produces the S-BART RV time series from HARPS spectra whose standard deviation is benchmarked against other pipelines) — https://github.com/iastro-pt/sBART
- **Telfit** (Telluric absorption model used to build the telluric template from the same observations, affecting the RV precision being compared) — https://github.com/kgullikson88/Telluric-Fitter
- **HARPS-TERRA** (Reference template-matching pipeline whose RV standard deviation provides the baseline for comparison)
- **SERVAL** (Reference template-matching pipeline whose RV standard deviation provides the baseline for comparison)

## Examples

```
pip install SBART
```

## Evaluation signals

- The S-BART RV time-series text file exists with all default columns populated (RV in km/s corrected by DRIFT and SA)
- The standard deviation of the S-BART RVs is smaller than that obtained with HARPS-TERRA and SERVAL on the same data
- RV_orderwise_errors.png and DataRejection.png diagnostics are generated and pathological orders/data rejections can be inspected
- Sigma-clipping of RVs against the DRS estimate is applied so no DRS-outlier frames bias the scatter comparison

## Limitations

- Performance differences against other pipelines are attributed to differences in the construction of the stellar template and the handling of telluric features, so a fair comparison requires consistent template/telluric choices
- S-BART can only currently be applied to HARPS and ESPRESSO data
- Telfit may fail to download GDAS profiles for some reference frames, requiring a fallback for Telfit templates

## Evidence

- [abstract] S-BART yields smaller RV standard deviation than HARPS-TERRA and SERVAL on Barnard's star HARPS data: "the standard deviation of the RVs derived with S-BART is smaller than that obtained with the HARPS-TERRA and SERVAL pipelines"
- [abstract] Semi-Bayesian RV extraction via common RV shift and Laplace approximation: "a common RV shift is assumed to describe the difference between each spectral order of a given stellar spectrum and a template built from the available observations."
- [methods] Frame filtering conditions in SBART: "Added a new condition to check order-wise SNR"
- [methods] RV output columns include instrumental drift and secular acceleration corrections: "RV measurement, in km/s, after being corrected by the instrumental drift (see DRIFT column description)"
- [abstract] S-BART applicability limited to HARPS and ESPRESSO: [evidence span withheld — anchor status: not_found]
