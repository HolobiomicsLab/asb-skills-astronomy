---
name: training-set-size-sweep-analysis
description: Use when you must determine the minimum number of training galaxies needed
  for a photo-z model (gamma-family GLM on PCA-projected magnitudes) to reach a target
  catastrophic outlier rate on a fixed test set.
when_to_use_negative:
- When the goal is photo-z point accuracy (bias/RMSE) rather than the training-size
  dependence of the catastrophic outlier fraction — use a k-fold cross-validation
  under/over-estimation study instead, which the paper explicitly declares out of
  scope
- When the input is not a photometric catalogue with known spectroscopic redshifts
  to define the |z_phot - z_spec| > 0.10 outlier metric
- When the data cannot support a fixed held-out test set; sweeping sizes against a
  changing test set invalidates the comparison
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: R
  role: Fit the gamma-family GLM with log link via iteratively reweighted least squares
    and run the training-size sweep
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
- name: Python
  role: Alternative implementation of the PCA projection and GLM photo-z pipeline
    (CosmoPhotoz Python package)
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
- name: CosmoPhotoz
  role: Package providing the robust PCA + gamma GLM photo-z workflow used for the
    sweep
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
- name: R/Shiny
  role: Interactive web application alternative for running the GLM photo-z estimation
  repo: https://cosmostatisticsinitiative.shinyapps.io/Cosmo Photoz
- name: statsmodels
  role: Python GLM backend referenced in the paper's discussion of implementations
  repo: https://github.com/statsmodels
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.002
    title: 'The overlooked potential of Generalized Linear Models in astronomy-II:
      Gamma regression and photometric redshifts'
  source_content_hash: sha256:37126df7d29b4282deede06471a71605f0f2795337fe7fbad3c1fbe019ab9b30
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/COINtoolbox/COSMOPhotoz
  tools:
  - CosmoPhotoz
  - Python
  - R
  - R/Shiny
  - statsmodels
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# training-set-size-sweep-analysis

## Summary

A method for quantifying how the catastrophic error (outlier fraction) of a gamma-family GLM photometric redshift estimate depends on training-set size, sweeping sizes from 500 to 10,000 galaxies to find the minimum size achieving a target error rate (e.g. < 5.937%).

## When to use

Use when you must determine the minimum number of training galaxies needed for a photo-z model (gamma-family GLM on PCA-projected magnitudes) to reach a target catastrophic outlier rate on a fixed test set.

## When NOT to use

- When the goal is photo-z point accuracy (bias/RMSE) rather than the training-size dependence of the catastrophic outlier fraction — use a k-fold cross-validation under/over-estimation study instead, which the paper explicitly declares out of scope
- When the input is not a photometric catalogue with known spectroscopic redshifts to define the |z_phot - z_spec| > 0.10 outlier metric
- When the data cannot support a fixed held-out test set; sweeping sizes against a changing test set invalidates the comparison

## Inputs

- PHAT0 multi-wavelength photometry catalogue (magnitudes plus spectroscopic redshifts)
- Random train/test split (training sample at least 10% of galaxies; paper used 1:9 yielding 33,904 train / 135,616 test)
- Robust PCA basis computed on the complete data set at a ~99.5% cumulative variance threshold
- GLM formula with interaction terms: zphot ~ PC1*PC2*...*PC4 + C

## Outputs

- results_training_size_sweep.csv containing 20 rows (training sizes 500 to 10,000, step 500) with catastrophic error per size
- Identified minimum training size whose catastrophic error is below the 5.937% threshold
- Per-size fitted gamma-family GLM models and test-set photo-z predictions

## How to apply

Load the PHAT0 multi-wavelength photometry catalogue and randomly split it into training and test sets (training sample holding at least 10% of galaxies, e.g. the paper's 1:9 split). Run robust PCA on the complete data set (training plus test) with a cumulative variance threshold of ~99.5%, then project onto the first 4 PCs as in the CosmoPhotoz package example. For each training-set size from 500 to 10,000 in increments of 500, subsample the training set and fit a gamma-family GLM with a log link using the factorial formula zphot ~ PC1*PC2*...*PC4 + C via iteratively reweighted least squares. Predict photo-z for the fixed test set at each size and compute the catastrophic error with the outlier definition O = |z_phot - z_spec| > 0.10. Tabulate training size vs catastrophic error and identify the smallest size whose error falls below the 5.937% threshold, writing the table and optimum to results_training_size_sweep.csv.

## Related tools

- **R** (Fit the gamma-family GLM with log link via iteratively reweighted least squares and run the training-size sweep) — https://github.com/COINtoolbox/COSMOPhotoz/
- **Python** (Alternative implementation of the PCA projection and GLM photo-z pipeline (CosmoPhotoz Python package)) — https://github.com/COINtoolbox/COSMOPhotoz/
- **CosmoPhotoz** (Package providing the robust PCA + gamma GLM photo-z workflow used for the sweep) — https://github.com/COINtoolbox/COSMOPhotoz/
- **R/Shiny** (Interactive web application alternative for running the GLM photo-z estimation) — https://cosmostatisticsinitiative.shinyapps.io/Cosmo Photoz
- **statsmodels** (Python GLM backend referenced in the paper's discussion of implementations) — https://github.com/statsmodels

## Examples

```
fit <- glm(zphot ~ (PC1+PC2+PC3+PC4)^2 + C, family=Gamma(link="log"), data=train_sub); predict(fit, newdata=test_pcs)
```

## Evaluation signals

- Sweep table must contain exactly 20 rows covering training sizes 500 through 10,000 in steps of 500
- Reported optimum must be the smallest size whose recorded catastrophic error is below the 5.937% threshold
- Catastrophic error at each size must be computed with the outlier definition O = |z_phot - z_spec| > 0.10 on the same fixed test set across all sizes
- PCA basis must be computed once on the complete data set (training plus test) at the ~99.5% cumulative variance threshold and projected consistently for train and test subsets
- Results should be consistent with the paper's claim that low catastrophic errors are achievable with training sets of merely thousands of galaxies

## Limitations

- Slight differences exist between the R and Python implementations in how they solve the SVD matrices in the PCA step and how they minimise the GLM, so sweep results may differ slightly across platforms
- Investigating under/over-estimation of photo-z would require a k-fold cross-validation analysis which is out of the scope of this work
- Default gamma family and inverse power link may not be fully optimal in all situations; reported performance is tied to a specific low-end hardware/OS configuration (AMD Athlon X2 QL-64, 1.7 GB RAM, Ubuntu 10.04)
- Tests using a simple formulation without interaction terms are inadequate in capturing the complexity of the data

## Evidence

- [methods] GLM fitting method: [evidence span withheld — anchor status: not_found]
