---
name: catastrophic-outlier-rate-evaluation
description: Use when a fitted photo-z model (e.g., gamma-family GLM with log link
  on PC projections) must be scored by its catastrophic outlier fraction on a test
  set with spectroscopic redshifts, or when comparing model variants (number of PCs,
  training-set size) by outlier rate.
when_to_use_negative:
- When the goal is detecting systematic under/over-estimation bias of photo-z — the
  paper states this 'would require a k-fold cross-validation analysis which is out
  of the scope of this work'
- When the test sample lacks spectroscopic redshifts, since z_spec is required for
  |z_phot - z_spec|
- When a per-redshift-bin scatter or heteroscedasticity diagnostic is needed rather
  than a single global outlier fraction
edam_operation: http://edamontology.org/operation_0320
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Fit the gamma-family GLM with log link via iteratively reweighted least squares
    and compute the catastrophic error fraction on the test set
- name: Python
  role: Predict photo-zs and compute O(a)/O(b) outlier fractions from PC projections
    and the best-fit GLM (statsmodels)
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
- name: CosmoPhotoz
  role: Package implementing GLM photo-z estimation whose predictions are evaluated
    with the catastrophic outlier rate metric
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_006
  - task_007
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.002
    title: 'The overlooked potential of Generalized Linear Models in astronomy-II:
      Gamma regression and photometric redshifts'
  source_content_hash: sha256:01922d2f65b17fd8b2c6c0d7fd7c13afaa8bf50977c80256bab32f5450181bac
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/COINtoolbox/COSMOPhotoz
  tools:
  - CosmoPhotoz
  - Python
  - R
  edam_operation: http://edamontology.org/operation_0320
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# catastrophic-outlier-rate-evaluation

## Summary

Quantifies photometric-redshift quality using the catastrophic error (outlier rate/fraction) metric, defined as the fraction of test galaxies with |z_phot - z_spec| > 0.10 (definition O(a)) or |z_phot - z_spec|/(1+z_spec) > 0.10 (definition O(b)). This is the primary quality metric used to benchmark gamma-family GLM photo-z fits on PHAT0 and SDSS DR10 data.

## When to use

Use when a fitted photo-z model (e.g., gamma-family GLM with log link on PC projections) must be scored by its catastrophic outlier fraction on a test set with spectroscopic redshifts, or when comparing model variants (number of PCs, training-set size) by outlier rate.

## When NOT to use

- When the goal is detecting systematic under/over-estimation bias of photo-z — the paper states this 'would require a k-fold cross-validation analysis which is out of the scope of this work'
- When the test sample lacks spectroscopic redshifts, since z_spec is required for |z_phot - z_spec|
- When a per-redshift-bin scatter or heteroscedasticity diagnostic is needed rather than a single global outlier fraction

## Inputs

- Test-set table with predicted photo-zs (z_phot) from the fitted gamma GLM and spectroscopic redshifts (z_spec)
- PC projections of the test data plus the best-fit GLM (R glm or Python statsmodels) used to generate predictions
- Training/test split of PHAT0 (1:9 ratio) or SDSS DR10 (1,347,640 galaxies, z = 0-1.0, u'g'r'i'z' magnitudes) catalogues

## Outputs

- Catastrophic outlier rate O(a) = |z_phot - z_spec| > 0.10 (fraction or percentage)
- Catastrophic outlier rate O(b) = |z_phot - z_spec|/(1+z_spec) > 0.10 for real-data (dereddened) comparisons
- Tabulated trade-off results (e.g., outlier rate vs. number of PCs or training-set size, as in results_training_size_sweep.csv or sdss_photoz_metrics.csv)

## How to apply

Split the catalogue into training and test sets with the training sample holding at least 10% of the galaxies (the paper used a 1:9 split for PHAT0: 33,904 training and 135,616 test galaxies). After fitting the gamma-family GLM (log link) on robust-PCA-derived PCs (cumulative variance threshold ~99.5%) and predicting photo-z for the test set, compute the catastrophic error as the fraction of test objects with |z_phot - z_spec| > 0.10 (definition O(a)); for SDSS-style analyses also compute O(b) = |z_phot - z_spec|/(1+z_spec) > 0.10 on dereddened magnitudes. Compare the resulting rates against known benchmarks to validate the fit: PHAT0 gamma GLM gives O(a) = 4.4% (6 PCs), 2.5% (8 PCs), 1.4% (10 PCs), while SDSS DR10 gives O(a) = 8.087% (4 PCs) and 6.439% (5 PCs). The metric can also serve as the target variable in sweeps, e.g., finding the minimum training-set size (500 to 10,000 galaxies in steps of 500) whose catastrophic error falls below a threshold such as 5.937%.

## Related tools

- **R** (Fit the gamma-family GLM with log link via iteratively reweighted least squares and compute the catastrophic error fraction on the test set)
- **Python** (Predict photo-zs and compute O(a)/O(b) outlier fractions from PC projections and the best-fit GLM (statsmodels)) — https://github.com/COINtoolbox/COSMOPhotoz/
- **CosmoPhotoz** (Package implementing GLM photo-z estimation whose predictions are evaluated with the catastrophic outlier rate metric) — https://github.com/COINtoolbox/COSMOPhotoz/

## Examples

```
outlier_rate = (abs(z_phot - z_spec) > 0.10).mean() * 100  # O(a), PHAT0/SDSS test set
```

## Evaluation signals

- Reproduced PHAT0 table matches O(a) = 4.4% (6 PCs), 2.5% (8 PCs), 1.4% (10 PCs) with fitting times ~170 s, 1200 s, 4948 s
- Reproduced SDSS DR10 values match O(a) = 8.087% (4 PCs) and 6.439% (5 PCs), with dereddened O(b) = 2.90 (4 PCs) and 2.25 (5 PCs)
- Both O(a) and O(b) definitions are reported when evaluating real (dereddened) data, as required by the SDSS validation step
- In a training-size sweep, the sweep table has 20 rows (500-10,000 galaxies, step 500) and the reported optimum is the smallest size with catastrophic error below 5.937%
- Metric is computed on the held-out test set only (e.g., the 135,616-galaxy PHAT0 test split), not on training galaxies

## Limitations

- The paper reports results on specific hardware (AMD Athlon X2 Dual-Core QL-64, 1.7 GB RAM, Ubuntu 10.04), so fitting times scale differently on other machines
- Slight differences exist between how the R and Python codes solve the SVD matrices in the PCA step and minimize the GLM, so rates may differ slightly between implementations
- The metric does not distinguish under- from over-estimation of photo-z; that requires k-fold cross-validation, which the paper leaves out of scope

## Evidence

- [methods] catastrophic error metric definition: [evidence span withheld — anchor status: not_found]
- [abstract] reported precision levels: "We obtain fits that result in catastrophic outlier rates as low as ∼ 1% for simulated and ∼ 2% for real data."
- [results] PC variance threshold used before evaluating outliers: "Decomposition by PCA showed that at least 6 components needed to be used to ensure that 99.5% of the variance was retained"
- [readme] CosmoPhotoz package scope: "CosmoPhotoz is a package that determines photometric redshifts from galaxies utilising their magnitudes. The method utilises Generalized Linear Models"
