---
name: median-rv-precision-benchmarking
description: 'Use when comparing RV precision of two extraction methods (e.g. template
  matching vs CCF) on the same spectra: you have per-observation RV uncertainty columns
  (RVc_ERR, DRIFT_ERR) and need a headline median-uncertainty metric plus a method-to-method
  ratio.'
when_to_use_negative:
- You need RV scatter/variability of a time series (e.g. planetary signal amplitudes)
  rather than per-observation formal uncertainties — the paper defers full planetary
  and activity signal analysis to future work.
- Your observations have drift already corrected at the DRS level (DRIFT_ERR = 'nan'),
  so no drift propagation step is needed and the quadrature addition would be wrong.
- You are comparing against HARPS-TERRA or SERVAL on HARPS archival data (e.g. Barnard's
  star) rather than benchmarking S-BART vs CCF on ESPRESSO data.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: S-BART
  role: Produces the template-matching RV time series and per-observation RVc_ERR
    and DRIFT_ERR uncertainty columns being benchmarked
  repo: https://github.com/iastro-pt/sBART
- name: ESO official pipeline
  role: Supplies the reference CCF RV uncertainties for the same ESPRESSO observations
- name: ESPRESSO
  role: Instrument whose GTO observations (33 targets; 309 observations of 16 M-type
    targets) form the benchmark dataset
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202142262
    title: A novel framework for semi-Bayesian radial velocities through template
      matching
  source_content_hash: sha256:d01992a4a024f47fdccb88482804712924281a608933925859f38e09e64796ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iastro-pt/sBART
  tools:
  - ESO official pipeline
  - ESPRESSO
  - S-BART
  - SBART
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# median-rv-precision-benchmarking

## Summary

Benchmark the per-observation RV uncertainties of a template-matching pipeline (S-BART) against a reference method (the ESO CCF pipeline) by propagating drift errors and computing median uncertainties across a fixed observation set (309 ESPRESSO observations of 16 M-type targets), targeting the reported ~15 cm/s median uncertainty.

## When to use

Use when comparing RV precision of two extraction methods (e.g. template matching vs CCF) on the same spectra: you have per-observation RV uncertainty columns (RVc_ERR, DRIFT_ERR) and need a headline median-uncertainty metric plus a method-to-method ratio.

## When NOT to use

- You need RV scatter/variability of a time series (e.g. planetary signal amplitudes) rather than per-observation formal uncertainties — the paper defers full planetary and activity signal analysis to future work.
- Your observations have drift already corrected at the DRS level (DRIFT_ERR = 'nan'), so no drift propagation step is needed and the quadrature addition would be wrong.
- You are comparing against HARPS-TERRA or SERVAL on HARPS archival data (e.g. Barnard's star) rather than benchmarking S-BART vs CCF on ESPRESSO data.

## Inputs

- S-BART RV time-series .txt files from RV_step/individual_subInst/<instrument>/ with RVc_ERR [Km/s] and DRIFT_ERR columns
- CCF RV uncertainties from the ESO official pipeline for the same ESPRESSO observations
- Set of 309 ESPRESSO GTO observations of 16 M-type targets

## Outputs

- rv_uncertainty_comparison.csv tabulating S-BART and CCF median RV uncertainties and their ratio
- Median RV uncertainty value in cm/s (target: ~15 cm/s for S-BART)

## How to apply

Load the RV time-series .txt files produced by the S-BART RV routine under RV_step/individual_subInst/<instrument>/ and parse the RVc_ERR [Km/s] uncertainty column together with the DRIFT_ERR column. Where DRIFT_ERR is not 'nan' (i.e. the instrumental drift was not already corrected at the DRS level), add it in quadrature to the raw S-BART RV uncertainty to propagate the instrumental drift contribution. Restrict to the 309 ESPRESSO observations of the 16 M-type targets, convert from km/s to cm/s, and take the median uncertainty; apply the identical median computation to the CCF RV uncertainties from the ESO official pipeline for the same observations. Tabulate both median uncertainties side by side with their ratio, writing the result to rv_uncertainty_comparison.csv, and check the S-BART median reproduces ~15 cm/s and the comparison is consistent with the reported ~10% decrease in median RV scatter for M-type stars (~4% for K-type).

## Related tools

- **S-BART** (Produces the template-matching RV time series and per-observation RVc_ERR and DRIFT_ERR uncertainty columns being benchmarked) — https://github.com/iastro-pt/sBART
- **ESO official pipeline** (Supplies the reference CCF RV uncertainties for the same ESPRESSO observations)
- **ESPRESSO** (Instrument whose GTO observations (33 targets; 309 observations of 16 M-type targets) form the benchmark dataset)

## Examples

```
pip install SBART
```

## Evaluation signals

- S-BART median RV uncertainty over the 309 observations of the 16 M-type targets reproduces the reported ~15 cm/s (after km/s-to-cm/s conversion and drift-error propagation)
- The S-BART-to-CCF median ratio is consistent with the reported ~10% decrease in median RV scatter for M-type stars (~4% for K-type)
- Every observation where DRIFT_ERR is 'nan' is excluded from the quadrature drift-propagation step, not treated as zero
- The comparison CSV contains both methods computed over the identical 309-observation set, so the ratio is like-for-like

## Limitations

- Drift uncertainty handling is conditional: DRIFT_ERR is 'nan' when the drift was already corrected at the DRS level, so the quadrature addition applies only where it is numeric.
- The NZP result (weighted scatter below ~0.7 m/s) should be taken as an upper limit on the RV precision attainable with ESPRESSO data, not a target value.
- Performance differences between methods are attributed to template construction and telluric handling, so benchmark results may not transfer to other instruments or telluric regimes.

## Evidence

- [abstract] S-BART achieves a median uncertainty of ~15 cm/s over 309 ESPRESSO observations: "S-BART achieves a median uncertainty of ~15 cm s−1over 309 observations of 16 targets"
- [abstract] Median RV scatter decreased ~10% for M-type stars vs CCF: "We find a decrease in the median RV scatter of ~10 and ~4% for M- and K-type stars, respectively."
- [abstract] Comparing S-BART to the CCF method in ESO's official pipeline: [evidence span withheld — anchor status: not_found]
- [methods] RVc_ERR is the drift-corrected RV measurement in km/s: "RV measurement, in km/s, after being corrected by the instrumental drift (see DRIFT column description)"
- [methods] sBART is pip-installable for reproduction: "SBART can be pip-installed:

    pip install SBART"
