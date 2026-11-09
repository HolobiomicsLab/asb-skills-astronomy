---
name: train-test-projection-consistency
description: Use when photometric magnitudes must be transformed into principal components
  as GLM covariates and training and test galaxies risk being decomposed on different
  bases, skewing photo-z predictions.
when_to_use_negative:
- The test set must remain fully blind at fitting time — this method deliberately
  fits robust PCA on training and test data together, so it violates strict blind-validation
  protocols.
- Covariates are already expressed in a shared, consistent basis (e.g. precomputed
  PCs or colours), so a new decomposition of the combined magnitudes would be redundant.
- A simple formulation without interaction terms is adequate — the paper notes such
  simple formulations are 'inadequate in capturing the complexity of the data', but
  if your task only needs plain linear covariates the full projection-plus-crossing
  machinery is overkill.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: CosmoPhotoz
  role: Reference R/Python/Shiny package implementing the robust PCA + gamma GLM photo-z
    pipeline with consistent train/test projection
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
- name: statsmodels
  role: Python library for fitting the gamma-family log-link GLM by iteratively re-weighted
    least squares on the projected covariates
  repo: https://github.com/statsmodels
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.002
    title: 'The overlooked potential of Generalized Linear Models in astronomy-II:
      Gamma regression and photometric redshifts'
  source_content_hash: sha256:16a2f2294ab5d177b675538a893d871ddf676603770d1338224887fa671df790
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/COINtoolbox/COSMOPhotoz
  tools:
  - CosmoPhotoz
  - Python
  - R
  - statsmodels
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# train-test-projection-consistency

## Summary

Ensures that PCA covariates used for gamma-GLM photometric redshift regression are computed on a single consistent basis: robust PCA is fitted on the COMPLETE magnitude data set (training and test galaxies together), and the test set is scored by projection onto that stored basis rather than by a separate decomposition.

## When to use

Use when photometric magnitudes must be transformed into principal components as GLM covariates and training and test galaxies risk being decomposed on different bases, skewing photo-z predictions.

## When NOT to use

- The test set must remain fully blind at fitting time — this method deliberately fits robust PCA on training and test data together, so it violates strict blind-validation protocols.
- Covariates are already expressed in a shared, consistent basis (e.g. precomputed PCs or colours), so a new decomposition of the combined magnitudes would be redundant.
- A simple formulation without interaction terms is adequate — the paper notes such simple formulations are 'inadequate in capturing the complexity of the data', but if your task only needs plain linear covariates the full projection-plus-crossing machinery is overkill.

## Inputs

- Combined photometric catalogue of galaxy magnitudes (training + test splits, e.g. PHAT0, PHAT1, SDSS DR10)
- Spectroscopic redshifts for the training split only
- Choice of number of PCs via cumulative variance threshold (~99.5%)

## Outputs

- Fitted robust PCA basis fitted on the complete training+test data set
- PC projections of the test data set
- Best-fit GLM model object (e.g. fitted_glm.pkl or .RDS)
- Per-galaxy z_phot predictions for the test set (e.g. photoz_predictions.csv)

## How to apply

Load magnitudes from both the training split (with spectroscopic redshifts) and the test split, then run robust principal component analysis (Candes et al. 2011-style) on the complete data set, training and test combined, so PCs are not dominated by either sample. Choose the number of components with a cumulative total-variance threshold of ~99.5% (the paper's PHAT0 case retained at least 6 components). Store the fitted PCA basis, fit the gamma-family, log-link GLM (zphot ~ PC1 + ... crossed with interactions, fitted by iteratively re-weighted least squares) on the projected training covariates, then compute test predictions by projecting test magnitudes onto that same stored basis and applying the best-fit GLM. The rationale is that a basis refitted on test data alone would give covariates in an inconsistent coordinate system and break the regression mapping.

## Related tools

- **CosmoPhotoz** (Reference R/Python/Shiny package implementing the robust PCA + gamma GLM photo-z pipeline with consistent train/test projection) — https://github.com/COINtoolbox/COSMOPhotoz/
- **statsmodels** (Python library for fitting the gamma-family log-link GLM by iteratively re-weighted least squares on the projected covariates) — https://github.com/statsmodels

## Examples

```
pca.fit(np.vstack([train_mag, test_mag])); Z_train = pca.transform(train_mag); Z_test = pca.transform(test_mag)
```

## Evaluation signals

- The PCA was fitted on the complete data set (training and test together), and the number of retained components reaches the ~99.5% cumulative variance threshold (PHAT0 needed at least 6 components).
- Test-set photo-z predictions are all valid positive continuous values, consistent with the gamma family's positive support.
- photoz_predictions.csv exists and contains one prediction per test-set galaxy, computed from the stored PCA basis projections and the best-fit GLM (not a refitted basis).
- Catastrophic outlier rate is in the range reported by the paper (~1% for simulated data, ~2% for real data).

## Limitations

- Slight differences exist between the R and Python implementations in how they solve the SVD matrices in the PCA step and how they minimise the GLM, so R and Python fits will not be bit-identical.
- Fitting PCA on the combined training+test data means the test magnitudes inform the decomposition, which is not a fully blind test protocol.
- The default gamma family and inverse power link may not be fully optimal in all situations if the user does not intervene.

## Evidence

- [methods] Robust PCA fitted on the complete data set: [evidence span withheld — anchor status: not_found]
- [methods] Test predictions use stored basis projections: [evidence span withheld — anchor status: not_found]
