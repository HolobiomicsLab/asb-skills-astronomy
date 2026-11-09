---
name: factorial-model-formula-specification
description: Use when fitting a gamma-family, log-link GLM to predict photometric
  redshifts from PCA-decomposed magnitudes and you must decide which covariates and
  interaction terms enter the model formula.
when_to_use_negative:
- When the goal is under/over-estimation diagnostics of photo-z — the article notes
  this would require a k-fold cross-validation analysis which was out of scope here
- When predictors are raw magnitudes rather than PCA components — the article's factorial
  specification operates on the principal components of the observed magnitude set
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R
  role: glm() fits the factorial-formula gamma GLM by iteratively re-weighted least
    squares
  repo: https://github.com/COINtoolbox/CosmoPhotoz
- name: Python
  role: statsmodels GLM fits the same factorial formula in the Python implementation
    of CosmoPhotoz
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
- name: statsmodels
  role: Python GLM engine used for the IRLS fit of the crossed-PC formula
  repo: https://github.com/statsmodels
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.002
    title: 'The overlooked potential of Generalized Linear Models in astronomy-II:
      Gamma regression and photometric redshifts'
  source_content_hash: sha256:e76f929ac2a41ef116bbdc659d0b356e23da5cf0e587adc5d7ac0947a169ba0c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/COINtoolbox/CosmoPhotoz
  tools:
  - Python
  - R
  - statsmodels
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# factorial-model-formula-specification

## Summary

Specify the gamma-family GLM photo-z model with R-style factorial formula syntax, e.g. zphot ~ PC1 + PC1*PC2*...*PCn + C, so the crossed PCs and their interaction terms are included alongside the intercept. Interaction terms are essential: simple formulations without them fail to capture the complexity of the photometric data.

## When to use

Use when fitting a gamma-family, log-link GLM to predict photometric redshifts from PCA-decomposed magnitudes and you must decide which covariates and interaction terms enter the model formula.

## When NOT to use

- When the goal is under/over-estimation diagnostics of photo-z — the article notes this would require a k-fold cross-validation analysis which was out of scope here
- When predictors are raw magnitudes rather than PCA components — the article's factorial specification operates on the principal components of the observed magnitude set

## Inputs

- Principal component scores from robust PCA of galaxy magnitudes (training and test sets, PCs retained to ~99.5% cumulative variance)
- Spectroscopic redshifts (zphot response) for the training split
- Photometric catalogues (e.g. PHAT0, PHAT1, SDSS DR10 magnitudes)

## Outputs

- Fitted gamma-family GLM model object with factorial formula (e.g. fitted_glm.pkl or .RDS)
- GLM coefficients for PC main effects and interaction terms
- Per-galaxy z_phot predictions for the test set (photoz_predictions.csv)

## How to apply

After running robust PCA (Candes et al. 2011-style) on the complete training+test photometric catalogue and retaining enough components for ~99.5% cumulative variance (6 components for PHAT0), express the model as a factorial formula: response zphot on the principal components with the crossing operator including all interaction terms (zphot ~ PC1 + PC1*PC2*...*PCn + C), under a gamma family reflecting the positive continuous support of redshifts. Fit this formula by iteratively re-weighted least squares, e.g. via statsmodels GLM in Python or glm in R. Do not drop the interaction terms — the article notes that tests using a simple formulation without interaction terms are inadequate in capturing the complexity of the data. Then use the fitted formula object to predict photo-z from the PCA projections of the test set.

## Related tools

- **R** (glm() fits the factorial-formula gamma GLM by iteratively re-weighted least squares) — https://github.com/COINtoolbox/CosmoPhotoz
- **Python** (statsmodels GLM fits the same factorial formula in the Python implementation of CosmoPhotoz) — https://github.com/COINtoolbox/COSMOPhotoz/
- **statsmodels** (Python GLM engine used for the IRLS fit of the crossed-PC formula) — https://github.com/statsmodels

## Examples

```
import statsmodels.api as sm; glm = sm.GLM(zphot, sm.add_constant(pd.DataFrame({'PC1':pc1,'PC2':pc2})), family=sm.families.Gamma(sm.families.links.log())).fit()
```

## Evaluation signals

- The fitted model includes interaction (crossed) terms between retained PCs, not only main effects
- PCA retained components meet the ~99.5% cumulative total-variance threshold (6 components for PHAT0)
- photoz_predictions.csv contains a valid positive continuous z_phot for every test-set galaxy, consistent with the gamma family's positive support
- Catastrophic outlier rates fall near the reported ~1% (simulated) / ~2% (real data) levels
- R and Python fits should agree up to known implementation differences in SVD solving and GLM minimisation

## Limitations

- Slight differences between the R and Python codes in how SVD matrices are solved in the PCA step and how the GLM is minimised are not fully reconciled
- Tests using a simple formulation without interaction terms are inadequate in capturing the complexity of the data
- Default gamma family and inverse power link may not be fully optimal in all situations if the user does not intervene

## Evidence

- [methods] interaction terms required: "We note that tests using a simple formulation without interaction terms are inadequate in capturing the complexity of the data."
- [results] PC count threshold: "Decomposition by PCA showed that at least 6 components needed to be used to ensure that 99.5% of the variance was retained"
- [results] gamma family fit: [evidence span withheld — anchor status: not_found]
