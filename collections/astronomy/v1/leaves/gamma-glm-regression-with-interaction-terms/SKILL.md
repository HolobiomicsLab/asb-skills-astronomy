---
name: gamma-glm-regression-with-interaction-terms
description: Use when predicting a positive, continuous target (e.g., galaxy photometric
  redshift z_phot) from dimensionality-reduced photometric features (principal components
  of u'g'r'i'z' magnitudes) and a fast, statistically grounded regression is needed.
when_to_use_negative:
- A simple additive formulation without interaction terms is used — the article states
  tests without interactions are 'inadequate in capturing the complexity of the data'
- The target variable is not strictly positive/continuous — the gamma family with
  log (or inverse power) link assumes this distributional form
- A k-fold cross-validation analysis of photo-z under/over-estimation is required
  — the authors state this is out of scope for this method
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: R
  role: Reference implementation of the gamma-family GLM with log link fitted by iteratively
    reweighted least squares (Appendix A)
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
- name: Python
  role: Alternative implementation of the PCA + gamma GLM photo-z pipeline (Appendix
    B); slight differences from R exist in SVD solving and GLM minimisation
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
- name: statsmodels
  role: Python library used to fit the GLM in the Python implementation
  repo: https://github.com/statsmodels
- name: CosmoPhotoz
  role: User-friendly package (R, R/Shiny webapp, Python) that determines photometric
    redshifts from galaxy magnitudes via GLMs
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.002
    title: 'The overlooked potential of Generalized Linear Models in astronomy-II:
      Gamma regression and photometric redshifts'
  source_content_hash: sha256:b23f9d318113b884bd41d9298681daaf53d1970a3ec8580ce67ae45902b4472d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/COINtoolbox/COSMOPhotoz
  tools:
  - CosmoPhotoz
  - Python
  - R
  - statsmodels
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# gamma-glm-regression-with-interaction-terms

## Summary

Fit a gamma-family Generalized Linear Model with a log link to predict photometric redshifts from PCA-reduced galaxy magnitudes, using a factorial formula with interaction (crossing) terms so the model captures non-linear structure in the photometry. Delivers catastrophic outlier rates as low as ~1–2% within seconds on a desktop machine.

## When to use

Use when predicting a positive, continuous target (e.g., galaxy photometric redshift z_phot) from dimensionality-reduced photometric features (principal components of u'g'r'i'z' magnitudes) and a fast, statistically grounded regression is needed.

## When NOT to use

- A simple additive formulation without interaction terms is used — the article states tests without interactions are 'inadequate in capturing the complexity of the data'
- The target variable is not strictly positive/continuous — the gamma family with log (or inverse power) link assumes this distributional form
- A k-fold cross-validation analysis of photo-z under/over-estimation is required — the authors state this is out of scope for this method

## Inputs

- Galaxy photometric catalogue (e.g., SDSS DR10: 1,347,640 galaxies, z = 0–1.0, u'g'r'i'z' magnitudes, retrievable via the SQL query in Appendix C)
- PCA principal components (robust PCA, ~99.5% cumulative variance threshold) for training and test sets
- Spectroscopic redshifts (z_spec) for the training sample

## Outputs

- Fitted gamma GLM (log link) with factorial formula including interaction terms
- Predicted photo-z values for the test set
- Catastrophic outlier fractions O(a) and O(b) (e.g., written to sdss_photoz_metrics.csv)

## How to apply

First run robust PCA on the complete data set (train + test) with the cumulative-variance threshold set to ~99.5% to fix the number of components (e.g., 4 PCs for SDSS DR10, 6 for PHAT0). Split the data randomly into training and test sets with the training sample holding at least 10% of galaxies (1:9 split for PHAT0). Fit the gamma-family GLM with log link via the factorial formula z_phot ~ PC1 * PC2 * ... * PCn + C, where the crossing operator includes interaction terms; simple additive formulations without interactions are explicitly inadequate for capturing the data complexity. The fit uses iteratively reweighted least squares, available in R or the Python statsmodels implementation, and predictions for the test set use the PC projections of the test data with the best-fit GLM from the training sample. Quality is judged via the catastrophic error fraction, O(a) = |z_phot - z_spec| > 0.10 and O(b) = |z_phot - z_spec|/(1 + z_spec) > 0.10; reported SDSS benchmarks are O(a) = 8.087% (4 PCs) improving to 6.439% (5 PCs with 50,000 training galaxies), with dereddened O(b) = 2.90% and 2.25%.

## Related tools

- **R** (Reference implementation of the gamma-family GLM with log link fitted by iteratively reweighted least squares (Appendix A)) — https://github.com/COINtoolbox/COSMOPhotoz/
- **Python** (Alternative implementation of the PCA + gamma GLM photo-z pipeline (Appendix B); slight differences from R exist in SVD solving and GLM minimisation) — https://github.com/COINtoolbox/COSMOPhotoz/
- **statsmodels** (Python library used to fit the GLM in the Python implementation) — https://github.com/statsmodels
- **CosmoPhotoz** (User-friendly package (R, R/Shiny webapp, Python) that determines photometric redshifts from galaxy magnitudes via GLMs) — https://github.com/COINtoolbox/COSMOPhotoz/

## Examples

```
glm(z_phot ~ PC1 * PC2 * PC3 * PC4, family = Gamma(link = "log"), data = train)  # factorial formula fit by IRLS
```

## Evaluation signals

- Reproduced O(a) for SDSS matches the reported values: 8.087% with 4 PCs and 6.439% with 5 PCs (50,000-galaxy training set)
- Dereddened O(b) values match 2.90% (4 PCs) and 2.25% (5 PCs)
- Number of retained PCs satisfies the cumulative variance threshold of ~99.5% (≥4 components for SDSS; PHAT0 required ≥6)
- All metrics are written to sdss_photoz_metrics.csv
- Fit completes within seconds on a single-core desktop even for large catalogues

## Limitations

- R and Python implementations differ slightly in how they solve the SVD matrices in the PCA step and how they minimise the GLM, so results are not perfectly reconciled between languages
- If the user does not intervene, the CosmoPhotoz package defaults to the gamma family and inverse power link, which may not be fully optimal in all situations
- Reported performance was tied to specific low-end hardware (AMD Athlon X2 Dual-Core QL-64, 1.7 GB RAM, Ubuntu 10.04); other configurations may differ
- The web-based (Shiny) interface requires no local installation but actual processing may be slower

## Evidence

- [methods] Factorial formula with interactions: [evidence span withheld — anchor status: not_found]
- [results] SDSS benchmark results: [evidence span withheld — anchor status: not_found]
