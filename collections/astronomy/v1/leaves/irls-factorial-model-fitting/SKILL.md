---
name: irls-factorial-model-fitting
description: Use when predicting galaxy photo-zs from magnitude-derived principal
  components and you need a fast regression that respects the positive, right-skewed
  redshift distribution via gamma regression with a log link.
when_to_use_negative:
- The target variable is not strictly positive and right-skewed (the gamma family
  with log link is tailored to redshift-like distributions)
- A simple additive formulation without interaction terms is acceptable — the paper
  shows such tests are inadequate for capturing the complexity of the data
- You need to characterise under/over-estimation of photo-z, which would require a
  k-fold cross-validation analysis that is out of the scope of this work
edam_operation: http://edamontology.org/operation_0320
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: R
  role: Fit the gamma-family GLM with log link and factorial formula (crossing operator)
    via iteratively reweighted least squares; reference implementation released in
    the CosmoPhotoz R package
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
- name: Python
  role: Alternative implementation of the gamma-regression photo-z fit in the CosmoPhotoz
    Python package
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
- name: Shiny
  role: Interactive web application (R/Shiny) where users can upload their data set
    and run the GLM photo-z estimation without local installation
  repo: https://cosmostatisticsinitiative.shinyapps.io/Cosmo Photoz
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.002
    title: 'The overlooked potential of Generalized Linear Models in astronomy-II:
      Gamma regression and photometric redshifts'
  source_content_hash: sha256:0224d25c19164ff66a89cb33a41740999a37d8da03ea0c947e7b9a9fa1359100
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/COINtoolbox/COSMOPhotoz
  tools:
  - Python
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_0320
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# irls-factorial-model-fitting

## Summary

Fit a gamma-family GLM with log link to photometric redshift estimation using a factorial R formula with interaction terms (crossing operator), fitted by iteratively reweighted least squares on principal components of observed magnitudes.

## When to use

Use when predicting galaxy photo-zs from magnitude-derived principal components and you need a fast regression that respects the positive, right-skewed redshift distribution via gamma regression with a log link.

## When NOT to use

- The target variable is not strictly positive and right-skewed (the gamma family with log link is tailored to redshift-like distributions)
- A simple additive formulation without interaction terms is acceptable — the paper shows such tests are inadequate for capturing the complexity of the data
- You need to characterise under/over-estimation of photo-z, which would require a k-fold cross-validation analysis that is out of the scope of this work

## Inputs

- Simulated or observed photometric catalogue (e.g. PHAT0, PHAT1, SDSS DR10) with magnitudes and spectroscopic redshifts
- Principal component projections of the magnitudes (robust PCA on training + test data, ~99.5% cumulative variance threshold)
- Train/test split definition (random 1:9 ratio, e.g. 33,904 training and 135,616 test galaxies for PHAT0)

## Outputs

- Fitted gamma-family GLM with log link over the factorial PC formula (coefficients from IRLS)
- Predicted photo-z values for the test set galaxies
- Catastrophic outlier rate O(a) with |z_phot - z_spec| > 0.10 and wall-clock fitting time

## How to apply

After robust PCA on the complete data set selects the number of PCs (cumulative variance threshold ~99.5%, e.g. at least 6 components for PHAT0), construct the factorial formula using R's crossing operator, e.g. zspec ~ PC1*PC2*...*PCn, so interaction terms are included — tests without interaction terms are inadequate for capturing the data complexity. Fit the model with the gamma family and log link using iteratively reweighted least squares, e.g. via glm(..., family=Gamma(link='log')) in R or the equivalent in the CosmoPhotoz package (R, Python, or Shiny webapp). Predict photo-z for the held-out test set (e.g. 135,616 PHAT0 galaxies from the 1:9 split) using the PC projections and the best-fit GLM. Evaluate quality with the catastrophic outlier rate O(a), defined by Delta_z(a) = |z_phot - z_spec| > 0.10, and record wall-clock fitting time to characterise the accuracy-runtime trade-off across PC counts (e.g. 6 PCs: 4.4% in 170 s; 8 PCs: 2.5% in 1200 s; 10 PCs: 1.4% in 4948 s).

## Related tools

- **R** (Fit the gamma-family GLM with log link and factorial formula (crossing operator) via iteratively reweighted least squares; reference implementation released in the CosmoPhotoz R package) — https://github.com/COINtoolbox/COSMOPhotoz/
- **Python** (Alternative implementation of the gamma-regression photo-z fit in the CosmoPhotoz Python package) — https://github.com/COINtoolbox/COSMOPhotoz/
- **Shiny** (Interactive web application (R/Shiny) where users can upload their data set and run the GLM photo-z estimation without local installation) — https://cosmostatisticsinitiative.shinyapps.io/Cosmo Photoz

## Examples

```
fit <- glm(zspec ~ PC1*PC2*PC3*PC4*PC5*PC6, family=Gamma(link="log"), data=train); pred <- predict(fit, newdata=test)
```

## Evaluation signals

- Reproduced PHAT0 table shows catastrophic outlier rates of ~4.4% (6 PCs), ~2.5% (8 PCs) and ~1.4% (10 PCs), matching section 5.3 of the paper
- Fitting times are in the reported range (approximately 170 s, 1200 s, 4948 s for 6, 8 and 10 PCs respectively on the paper's low-end laptop)
- PCA decomposition retains at least 99.5% of the variance in the chosen number of components (minimum 6 for PHAT0)
- Model formula includes interaction terms via the crossing operator (e.g. PC1*PC2*...*PCn), not a purely additive form
- Predicted photo-z values are strictly positive, consistent with the gamma family and log link

## Limitations

- Default gamma family and inverse power link may not be fully optimal in all situations if the user does not interfere
- Slight differences exist between the R and Python implementations in how they solve the SVD matrices in the PCA step and how they minimise the GLM
- Reported fitting times are tied to specific low-end hardware/OS (AMD Athlon X2 Dual-Core QL-64, 1.7 GB RAM, Ubuntu 10.04) and will vary on other machines
- The web-based Shiny interface requires no local installation but actual processing may be slower

## Evidence

- [methods] This formula was fit using iteratively re-weighted least squares.: [evidence span withheld — anchor status: not_found]
- [results] The PHAT0 data set was fit using the gamma family with a log link: [evidence span withheld — anchor status: not_found]
- [methods] Catastrophic outlier metric definition: [evidence span withheld — anchor status: not_found]
