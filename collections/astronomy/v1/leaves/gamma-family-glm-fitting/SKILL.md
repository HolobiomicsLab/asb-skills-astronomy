---
name: gamma-family-glm-fitting
description: Use when photometric redshifts (or any positive, continuous response)
  must be regressed on magnitude-derived principal components — e.g. PHAT0 or SDSS
  DR10 catalogues — using a gamma GLM with interaction terms fitted by IRLS.
when_to_use_negative:
- No training labels (spectroscopic redshifts) are available — the GLM requires a
  supervised training split with at least 10% of galaxies
- The goal is to assess under/over-estimation of photo-z via k-fold cross-validation,
  which the authors explicitly declare out of scope for this framework
- The response variable can be zero or negative — the gamma family's positive continuous
  support would be violated
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R
  role: glm() with family=Gamma(link='log') performs the iteratively re-weighted least
    squares fit of the factorial formula (CosmoPhotoz Appendix A is written in R)
  repo: https://github.com/COINtoolbox/CosmoPhotoz
- name: statsmodels
  role: Python GLM fitting (sm.GLM with Gamma family, log link) for the IRLS fit and
    test-set prediction (CosmoPhotoz Appendix B implementation)
  repo: https://github.com/statsmodels
- name: CosmoPhotoz
  role: User-friendly package implementing the full gamma GLM photo-z workflow in
    R, Python, and a Shiny web app
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
- name: Shiny
  role: Interactive web application where users can upload their data set and run
    the gamma GLM photo-z estimation without local installation
  repo: https://cosmostatisticsinitiative.shinyapps.io/Cosmo Photoz
provenance:
  source_task_ids:
  - task_004
  - task_007
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.002
    title: 'The overlooked potential of Generalized Linear Models in astronomy-II:
      Gamma regression and photometric redshifts'
  source_content_hash: sha256:8a90ce830afea0e00f7057178f1f741d684a0cfdeea6c4a4ca5b2ba450340fa8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/COINtoolbox/CosmoPhotoz
  tools:
  - CosmoPhotoz
  - Python
  - R
  - Shiny
  - statsmodels
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# gamma-family-glm-fitting

## Summary

Fits a gamma-family generalized linear model with a log link (via iteratively re-weighted least squares) to predict photometric redshifts from robust-PCA components of galaxy magnitudes, exploiting the gamma distribution's positive support to reproduce the physical properties of the redshift distribution.

## When to use

Use when photometric redshifts (or any positive, continuous response) must be regressed on magnitude-derived principal components — e.g. PHAT0 or SDSS DR10 catalogues — using a gamma GLM with interaction terms fitted by IRLS.

## When NOT to use

- No training labels (spectroscopic redshifts) are available — the GLM requires a supervised training split with at least 10% of galaxies
- The goal is to assess under/over-estimation of photo-z via k-fold cross-validation, which the authors explicitly declare out of scope for this framework
- The response variable can be zero or negative — the gamma family's positive continuous support would be violated

## Inputs

- Galaxy photometric catalogue with observed magnitudes and spectroscopic redshifts for the training split (e.g. PHAT0train: 161,042 galaxies; PHAT0test: 8,478 galaxies, or SDSS DR10 subset)
- PCA-projected explanatory variables (PC1...PCn) computed on the combined training + test magnitudes with a ~99.5% cumulative variance threshold

## Outputs

- Fitted gamma-family GLM model object (e.g. fitted_glm.pkl or .RDS)
- photoz_predictions.csv with a positive continuous z_phot for each test-set galaxy
- Quality metrics: bias (delta-z mean), rms(delta-z), and catastrophic outlier fraction

## How to apply

First run robust principal component analysis (Candes et al. 2011-style) on the COMPLETE data set (training plus test magnitudes together), selecting components by a cumulative total-variance threshold of ~99.5% (6 components for PHAT0). Then fit a gamma-family GLM with log link using the factorial formula zphot ~ PC1 * PC2 * ... * PCn + C, where the crossing operator includes interaction terms because simple formulations without interactions are inadequate for capturing the data complexity; the fit uses iteratively re-weighted least squares via R's glm or Python's statsmodels GLM. Project the test data onto the stored PCA basis and compute predicted photo-z for every test galaxy from the best-fit GLM. The gamma family is chosen to reflect the positive, continuous nature of redshifts, so every prediction must be positive. Save the fitted model object (e.g. fitted_glm.pkl or RDS) and per-galaxy photoz_predictions.csv, then evaluate bias, rms(delta-z), and the catastrophic outlier fraction |z_phot - z_spec|/(1 + z_spec) > 0.10.

## Related tools

- **R** (glm() with family=Gamma(link='log') performs the iteratively re-weighted least squares fit of the factorial formula (CosmoPhotoz Appendix A is written in R)) — https://github.com/COINtoolbox/CosmoPhotoz
- **statsmodels** (Python GLM fitting (sm.GLM with Gamma family, log link) for the IRLS fit and test-set prediction (CosmoPhotoz Appendix B implementation)) — https://github.com/statsmodels
- **CosmoPhotoz** (User-friendly package implementing the full gamma GLM photo-z workflow in R, Python, and a Shiny web app) — https://github.com/COINtoolbox/COSMOPhotoz/
- **Shiny** (Interactive web application where users can upload their data set and run the gamma GLM photo-z estimation without local installation) — https://cosmostatisticsinitiative.shinyapps.io/Cosmo Photoz

## Examples

```
glm(zphot ~ PC1*PC2*PC3*PC4, family = Gamma(link = "log"), data = train)  # R: IRLS fit of the factorial gamma GLM; Python equivalent: sm.GLM(y, X, family=sm.families.Gamma(sm.families.links.Log())).fit()
```

## Evaluation signals

- On PHAT0 (161,042 train / 8,478 test galaxies, 10 PCs reported in the reference reproduction), the fit should yield bias ≈ 0.033, rms(delta-z) ≈ 0.025, and catastrophic outlier rate ≈ 1.367% (~1.4%) within small numerical tolerance
- PCA variance check: the retained components reach the ~99.5% cumulative variance threshold (at least 6 components for PHAT0)
- Every predicted z_phot is strictly positive and continuous, consistent with the gamma family's support
- photoz_predictions.csv exists and contains one valid prediction per test-set galaxy
- Catastrophic outlier definition applied consistently: |z_phot - z_spec|/(1 + z_spec) > 0.10

## Limitations

- Tests using a simple formulation without interaction terms are inadequate in capturing the complexity of the data — the factorial/crossing formula with interactions is required
- Slight differences exist between the R and Python implementations in how the SVD matrices are solved in the PCA step and how the GLM is minimised, so results are not fully reconciled across languages
- If the user does not intervene, the default gamma family and inverse power link are used, which may not be fully optimal in all situations
- Performance claims are tied to a specific low-end hardware/OS configuration (AMD Athlon X2 QL-64, 1.7 GB RAM, Ubuntu 10.04); web-based (Shiny) processing may be slower than a local installation

## Evidence

- [methods] Robust PCA on the complete dataset: [evidence span withheld — anchor status: not_found]
- [methods] IRLS fitting of the factorial formula: [evidence span withheld — anchor status: not_found]
- [results] Gamma family with log link on PHAT0: [evidence span withheld — anchor status: not_found]
