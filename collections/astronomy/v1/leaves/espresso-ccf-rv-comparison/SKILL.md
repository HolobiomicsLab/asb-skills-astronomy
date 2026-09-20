---
name: espresso-ccf-rv-comparison
description: Use when comparing template-matching RV precision against DRS CCF uncertainties
  for ESPRESSO M-type spectra, or when validating an S-BART RV run against the paper's
  ~15 cm/s median uncertainty claim.
when_to_use_negative:
- The goal is analyzing planetary or activity signals in the RV time-series — the
  paper explicitly leaves 'a full analysis of the planetary and activity signals present
  in the considered datasets for future work'.
- The comparison target is K-type stars, where the expected median RV scatter decrease
  versus the CCF is ~4%, not the ~10% M-type benchmark.
- Validating S-BART against other template-matching pipelines on HARPS data (e.g.
  HARPS-TERRA or SERVAL on Barnard's star), which is a separate validation stage of
  the workflow.
tools:
- name: S-BART
  role: Produces the template-matching RV time-series and per-observation uncertainty
    columns (RVc_ERR [Km/s], DRIFT_ERR) that are aggregated and compared
  repo: https://github.com/iastro-pt/sBART
- name: ESO official pipeline
  role: Source of the reference CCF RV uncertainties for the same ESPRESSO observations
    used as the comparison baseline
- name: ESPRESSO
  role: Instrument whose GTO observations (33 targets, 309 observations of 16 M-type
    targets) form the evaluation dataset
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202142262
    title: A novel framework for semi-Bayesian radial velocities through template
      matching
  source_content_hash: sha256:aa0071833ec3d033eb2cecaada562c11e20b6abd9ba6dd1380e07cc2de3d9511
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iastro-pt/sBART
  tools:
  - ESO official pipeline
  - ESPRESSO
  - S-BART
  - SBART
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# espresso-ccf-rv-comparison

## Summary

Benchmark S-BART template-matching radial velocities against the CCF method from the ESO official pipeline for ESPRESSO observations, verifying the reported ~15 cm/s median RV uncertainty over 309 observations of 16 M-type targets and the ~10% decrease in median RV scatter for M-type stars.

## When to use

Use when comparing template-matching RV precision against DRS CCF uncertainties for ESPRESSO M-type spectra, or when validating an S-BART RV run against the paper's ~15 cm/s median uncertainty claim.

## When NOT to use

- The goal is analyzing planetary or activity signals in the RV time-series — the paper explicitly leaves 'a full analysis of the planetary and activity signals present in the considered datasets for future work'.
- The comparison target is K-type stars, where the expected median RV scatter decrease versus the CCF is ~4%, not the ~10% M-type benchmark.
- Validating S-BART against other template-matching pipelines on HARPS data (e.g. HARPS-TERRA or SERVAL on Barnard's star), which is a separate validation stage of the workflow.

## Inputs

- S-BART RV time-series .txt files under RV_step/individual_subInst/<instrument>/ containing 'RVc_ERR [Km/s]' and 'DRIFT_ERR' columns
- CCF RV uncertainties from the ESO official pipeline (ESPRESSO DRS) for the same 309 observations
- ESPRESSO GTO dataset covering 33 targets, of which 16 are M-type with 309 observations

## Outputs

- Median S-BART RV uncertainty in cm/s over the 309 M-type observations
- Median CCF RV uncertainty in cm/s over the same observations
- rv_uncertainty_comparison.csv tabulating S-BART vs CCF median uncertainties and their ratio

## How to apply

Load the RV time-series .txt files produced by the S-BART RV routine under RV_step/individual_subInst/<instrument>/, parsing the 'RVc_ERR [Km/s]' uncertainty column together with the 'DRIFT_ERR' column. Where DRIFT_ERR is not 'nan' (i.e. the instrumental drift was not already corrected at the DRS level), propagate it in quadrature with the raw S-BART RV uncertainty; where it is 'nan', use the RVc_ERR value as-is. Convert uncertainties from km/s to cm/s and compute the median across all 309 observations of the 16 M-type ESPRESSO GTO targets. Apply the identical median-uncertainty computation to the CCF RV uncertainties delivered by the ESO official pipeline for the same observations. Tabulate the S-BART and CCF median uncertainties side by side with their ratio and write the table to rv_uncertainty_comparison.csv; the S-BART median should land near ~15 cm/s and the ratio should be consistent with the reported ~10% decrease in median RV scatter for M-type stars.

## Related tools

- **S-BART** (Produces the template-matching RV time-series and per-observation uncertainty columns (RVc_ERR [Km/s], DRIFT_ERR) that are aggregated and compared) — https://github.com/iastro-pt/sBART
- **ESO official pipeline** (Source of the reference CCF RV uncertainties for the same ESPRESSO observations used as the comparison baseline)
- **ESPRESSO** (Instrument whose GTO observations (33 targets, 309 observations of 16 M-type targets) form the evaluation dataset)

## Examples

```
pip install SBART
```

## Evaluation signals

- Median S-BART RV uncertainty across the 309 observations of 16 M-type targets reproduces ~15 cm/s after the km/s-to-cm/s conversion
- The S-BART-to-CCF median comparison is consistent with the reported ~10% decrease in median RV scatter for M-type stars
- Quadrature propagation of DRIFT_ERR is applied only to rows where the drift was not already corrected at the DRS level (DRIFT_ERR not 'nan')
- rv_uncertainty_comparison.csv exists with side-by-side S-BART and CCF median uncertainty columns plus their ratio
- Observation and target counts match the ESPRESSO GTO evaluation set: 309 observations across 16 M-type targets

## Limitations

- DRIFT_ERR is 'nan' when the drift was already corrected at the DRS level, so naive quadrature addition over all rows would double-count the drift correction
- The weighted nightly zero point (NZP) scatter of ESPRESSO was estimated below ~0.7 m/s and 'should be taken as an upper limit on the RV precision attainable with ESPRESSO data', bounding interpretation of cm/s-level differences
- S-BART attributes its performance edge over the CCF to 'differences in the construction of the stellar template and the handling of telluric features', so the comparison reflects those pipeline choices rather than template matching alone
- The comparison covers precision/scatter only; full analysis of planetary and activity signals was deferred to future work

## Evidence

- [abstract] S-BART median RV uncertainty ~15 cm/s over 309 M-type observations: "S-BART achieves a median uncertainty of ~15 cm s−1over 309 observations of 16 targets"
- [abstract] ~10% decrease in median RV scatter for M-type stars vs CCF: "We find a decrease in the median RV scatter of ~10 and ~4% for M- and K-type stars, respectively."
- [abstract] CCF baseline comes from the ESO official pipeline: [evidence span withheld — anchor status: not_found]
- [methods] S-BART is pip-installable, grounding the example invocation: "SBART can be pip-installed:

    pip install SBART"
