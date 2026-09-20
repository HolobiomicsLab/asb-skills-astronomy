---
name: photometric-redshift-error-metrics
description: Use when predicted photometric redshifts for a galaxy test set with known
  spectroscopic redshifts must be scored via bias, rms(delta-z), and catastrophic
  outlier rate (e.g. validating a GLM photo-z fit on PHAT0/PHAT1/SDSS data).
when_to_use_negative:
- The test sample has no spectroscopic redshifts, so delta-z and the outlier criterion
  cannot be computed
- The goal is investigating under/over-estimation of photo-z via k-fold cross-validation,
  which the paper explicitly declares out of scope for the GLM framework
- The model was fit with a simple formulation without interaction terms, which the
  paper notes is inadequate at capturing the data complexity — metrics on such a fit
  would not be comparable
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: R
  role: Fit the gamma-family GLM and compute the bias, rms(delta-z), and catastrophic
    outlier metrics on the PHAT0 test set
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
- name: Python
  role: Alternative implementation for PC projections, GLM prediction, and metric
    computation; slight differences exist versus R in SVD solving and GLM minimisation
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
- name: Shiny
  role: Web application front end of CosmoPhotoz where users can upload a data set
    and obtain photo-z estimates whose outlier rate can be inspected
  repo: https://cosmostatisticsinitiative.shinyapps.io/Cosmo Photoz
- name: seaborn
  role: Statistical visualization of photo-z quality (e.g. delta-z distributions)
    alongside the numeric metrics
  repo: https://github.com/mwaskom/seaborn
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.002
    title: 'The overlooked potential of Generalized Linear Models in astronomy-II:
      Gamma regression and photometric redshifts'
  source_content_hash: sha256:480b681f2ebe2caf8dd1d9707efd87a83ecdc48532a22d586e6f2aed5ae6382e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/COINtoolbox/COSMOPhotoz
  tools:
  - Python
  - R
  - Shiny
  - seaborn
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# photometric-redshift-error-metrics

## Summary

Computes the standard photometric redshift quality metrics — bias (mean of delta-z), rms(delta-z), and the catastrophic outlier fraction (|z_phot - z_spec|/(1 + z_spec) > 0.10) — on a test set with spectroscopic redshifts, as used to validate the gamma-family GLM photo-z fits on PHAT0 in the CosmoPhotoz workflow.

## When to use

Use when predicted photometric redshifts for a galaxy test set with known spectroscopic redshifts must be scored via bias, rms(delta-z), and catastrophic outlier rate (e.g. validating a GLM photo-z fit on PHAT0/PHAT1/SDSS data).

## When NOT to use

- The test sample has no spectroscopic redshifts, so delta-z and the outlier criterion cannot be computed
- The goal is investigating under/over-estimation of photo-z via k-fold cross-validation, which the paper explicitly declares out of scope for the GLM framework
- The model was fit with a simple formulation without interaction terms, which the paper notes is inadequate at capturing the data complexity — metrics on such a fit would not be comparable

## Inputs

- Predicted photometric redshifts (z_phot) for a test sample, e.g. the 8,478-object PHAT0 test set from the CosmoPhotoz package
- True spectroscopic redshifts (z_spec) for the same test objects
- Optionally the fitted gamma-family GLM (log link) and test-set PC projections used to generate z_phot

## Outputs

- Bias (mean of delta-z) as a scalar, e.g. ~0.033 for PHAT0
- rms(delta-z) as a scalar, e.g. ~0.025 for PHAT0
- Catastrophic outlier fraction (%) for |z_phot - z_spec|/(1 + z_spec) > 0.10, e.g. ~1.367% for PHAT0
- CSV file (phat0_glm_metrics.csv) containing the three named metric fields

## How to apply

After predicting photo-z for the test set using the principal component projections and the best-fit gamma-family GLM with log link, compute delta-z = z_phot - z_spec for each test object and report its mean (bias) and root-mean-square. Count as catastrophic outliers all objects with |z_phot - z_spec|/(1 + z_spec) > 0.10 and express their fraction as a percentage. For the PHAT0 benchmark this pipeline reproduces bias ~0.033, rms(delta-z) ~0.025, and a catastrophic outlier rate of ~1.367% (10 PCs; the paper's PCA retained at least 6 components for 99.5% of the variance). Write the three metrics to a CSV (e.g. phat0_glm_metrics.csv) with the named fields bias, rms, and catastrophic outlier rate so the result can be diffed against the published values.

## Related tools

- **R** (Fit the gamma-family GLM and compute the bias, rms(delta-z), and catastrophic outlier metrics on the PHAT0 test set) — https://github.com/COINtoolbox/COSMOPhotoz/
- **Python** (Alternative implementation for PC projections, GLM prediction, and metric computation; slight differences exist versus R in SVD solving and GLM minimisation) — https://github.com/COINtoolbox/COSMOPhotoz/
- **Shiny** (Web application front end of CosmoPhotoz where users can upload a data set and obtain photo-z estimates whose outlier rate can be inspected) — https://cosmostatisticsinitiative.shinyapps.io/Cosmo Photoz
- **seaborn** (Statistical visualization of photo-z quality (e.g. delta-z distributions) alongside the numeric metrics) — https://github.com/mwaskom/seaborn

## Examples

```
fit <- glm(zphot ~ PC1*PC2*PC3*PC4*PC5*PC6 + C, data=phat0_train, family=Gamma(link="log")); dz <- predict(fit, phat0_test) - phat0_test$zspec
```

## Evaluation signals

- Catastrophic outlier rate on the PHAT0 test set reproduces the reported value of ~1.367% (paper-level expectation: ~1% for simulated data) within numerical tolerance
- Bias and rms(delta-z) land near the published PHAT0 values (~0.033 and ~0.025, respectively)
- Output CSV contains exactly the three named metric fields (bias, rms(delta-z), catastrophic outlier fraction)
- Outlier definition matches the literature metric: |z_phot - z_spec|/(1 + z_spec) > 0.10
- The upstream PCA retained enough components (>=6) to keep ~99.5% of the variance, ensuring metrics are computed on the intended feature set

## Limitations

- Slight differences exist between the R and Python implementations in how the SVD matrices are solved in the PCA step and how the GLM is minimised, so metrics may differ marginally between platforms
- Investigating under/over-estimation of photo-z would require a k-fold cross-validation analysis, which is out of the scope of the published workflow
- Reported performance was tied to a specific low-end hardware/OS configuration (AMD Athlon X2 QL-64, 1.7 GB RAM, Ubuntu 10.04); runtime characteristics may not transfer
- The ~1% outlier rate applies to simulated data; real data (e.g. PHAT1) yields ~2%, so expected metric ranges differ by dataset type

## Evidence

- [methods] Catastrophic error metric definition: [evidence span withheld — anchor status: not_found]
- [methods] Prediction step feeding the metrics: [evidence span withheld — anchor status: not_found]
