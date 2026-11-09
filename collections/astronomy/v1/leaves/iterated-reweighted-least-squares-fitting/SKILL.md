---
name: iterated-reweighted-least-squares-fitting
description: Use when predicting positive, continuous responses such as galaxy photo-z
  from PCA-reduced u'g'r'i'z' magnitudes via a gamma GLM whose covariates must be
  estimated by likelihood maximisation with IRLS.
when_to_use_negative:
- When a simple additive formulation without interaction terms is proposed — tests
  show such formulations are inadequate in capturing the complexity of the data
- When the response can take non-positive values — the gamma family only supports
  positive continuous redshifts
- When you need to assess systematic under/over-estimation of photo-z — that requires
  k-fold cross-validation, which the authors explicitly left out of scope
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R glm
  role: Fits the gamma-family, log-link factorial formula by iteratively re-weighted
    least squares
- name: statsmodels
  role: Python GLM engine (sm.GLM with Gamma family) for the same IRLS fit
  repo: github.com/statsmodels
- name: CosmoPhotoz
  role: Packaged R/Python/Shiny implementation of the GLM photo-z method described
    in the paper
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.002
    title: 'The overlooked potential of Generalized Linear Models in astronomy-II:
      Gamma regression and photometric redshifts'
  source_content_hash: sha256:b66f9d0dc7c1fdf252efd24d461341d593e70a1389b48eff2067a1d6d98f8753
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/github.com/statsmodels
  tools:
  - CosmoPhotoz
  - Python
  - R
  - R glm
  - statsmodels
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# iterated-reweighted-least-squares-fitting

## Summary

Fits a gamma-family GLM with log link to photometric redshifts using iteratively re-weighted least squares (IRLS), maximising the model log-likelihood (Eqn. 10) with a factorial formula of principal components including interaction terms.

## When to use

Use when predicting positive, continuous responses such as galaxy photo-z from PCA-reduced u'g'r'i'z' magnitudes via a gamma GLM whose covariates must be estimated by likelihood maximisation with IRLS.

## When NOT to use

- When a simple additive formulation without interaction terms is proposed — tests show such formulations are inadequate in capturing the complexity of the data
- When the response can take non-positive values — the gamma family only supports positive continuous redshifts
- When you need to assess systematic under/over-estimation of photo-z — that requires k-fold cross-validation, which the authors explicitly left out of scope

## Inputs

- Training photometric catalogue with spectroscopic redshifts (e.g., SDSS DR10 u'g'r'i'z' magnitudes, z = 0-1.0)
- PCA-reduced explanatory variables (PC1...PCn, n chosen at ~99.5% cumulative variance)
- Test-set PC projections for prediction

## Outputs

- Best-fit gamma GLM model object (e.g., fitted_glm.pkl or .RDS)
- Per-galaxy positive continuous z_phot predictions for the test set (photoz_predictions.csv)
- Catastrophic outlier rates O(a) = |z_phot - z_spec| > 0.10 and O(b) = |z_phot - z_spec|/(1 + z_spec) > 0.10 (e.g., sdss_photoz_metrics.csv)

## How to apply

First run robust PCA (Candès et al. 2011-style) on the complete data set (train + test) with a cumulative-variance threshold of ~99.5%, carrying 4 or 5 PCs forward as explanatory variables. Then fit the factorial formula z_phot ~ PC1 * PC2 * ... * PCn + C (crossing operator includes interaction terms) with a gamma family and log link, using iteratively reweighted least squares — via R's glm() or Python's statsmodels GLM — to maximise the log-likelihood. Simple formulations without interaction terms are inadequate for capturing the data complexity. Project the test data onto the stored PCA basis and predict photo-z for every test galaxy; the gamma family enforces the positive support appropriate to redshifts. On SDSS DR10 this converges within ~10 seconds even on modest hardware, yielding O(a) = 8.087% (4 PCs) and 6.439% (5 PCs, 50,000-galaxy training set).

## Related tools

- **R glm** (Fits the gamma-family, log-link factorial formula by iteratively re-weighted least squares)
- **statsmodels** (Python GLM engine (sm.GLM with Gamma family) for the same IRLS fit) — github.com/statsmodels
- **CosmoPhotoz** (Packaged R/Python/Shiny implementation of the GLM photo-z method described in the paper) — https://github.com/COINtoolbox/COSMOPhotoz/

## Examples

```
import statsmodels.api as sm; fit = sm.GLM(z_phot, X, family=sm.families.Gamma(sm.families.links.log())).fit()
```

## Evaluation signals

- Reproduced O(a) outlier fractions match reported SDSS values: 8.087% (4 PCs) and 6.439% (5 PCs)
- Dereddened O(b) values match 2.90 (4 PCs) and 2.25 (5 PCs)
- Every z_phot prediction in photoz_predictions.csv is a valid positive continuous number (gamma-family support)
- Retained PCA variance meets the ~99.5% cumulative threshold before fitting
- Fit completes within seconds on a standard desktop (training sets of merely thousands of galaxies)

## Limitations

- Slight differences exist between the R and Python implementations in how they solve the SVD matrices in the PCA step and how they minimise the GLM
- The default gamma family and inverse power link may not be fully optimal in all situations if the user does not intervene
- Reported runtime performance was tied to a specific low-end configuration (AMD Athlon X2 QL-64, 1.7 GB RAM, Ubuntu 10.04)

## Evidence

- [methods] This formula was fit using iteratively re-weighted least squares.: [evidence span withheld — anchor status: not_found]
- [discussion] We note that there are slightly differences to how the R and Python codes solve the SVD matrices in the PCA step and also how they minimise the GLM.: [evidence span withheld — anchor status: not_found]
