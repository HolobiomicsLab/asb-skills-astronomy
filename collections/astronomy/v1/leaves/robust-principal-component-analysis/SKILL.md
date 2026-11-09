---
name: robust-principal-component-analysis
description: Use when magnitudes fed into a regression (e.g. photometric redshift
  GLM) are correlated and you need uncorrelated explanatory variables computed robustly
  on combined training+test data.
when_to_use_negative:
- When the covariates are already uncorrelated or physically independent inputs —
  PCA on the complete set would add unnecessary preprocessing and mixing between training
  and test samples.
- When the downstream model requires interpretable per-band coefficients — PCs of
  the magnitude set lose direct magnitude-level interpretation.
- When the goal is a simple formulation without interaction terms capturing data complexity
  — the paper notes such simple tests are inadequate, but if a strictly linear, band-wise
  model is required, PCs may not be the right explanatory variables.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Python
  role: Implementation language for the robust PCA + GLM pipeline (CosmoPhotoz package,
    scikit-learn/statsmodels ecosystem)
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
- name: R
  role: Implementation language for the robust PCA + GLM photo-z code (Appendix A
    of the paper)
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.002
    title: 'The overlooked potential of Generalized Linear Models in astronomy-II:
      Gamma regression and photometric redshifts'
  source_content_hash: sha256:383175c156bcbb38901e2834df1cf6c591edf8ba5ed859f99c79fc060d017ff4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/COINtoolbox/COSMOPhotoz
  tools:
  - Python
  - R
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# robust-principal-component-analysis

## Summary

Robust PCA (Candes et al. 2011-style) decorrelates galaxy magnitudes into principal components that serve as explanatory variables for a gamma-family, log-link GLM photo-z estimator. The number of retained components is chosen by a cumulative total-variance threshold of ~99.5%.

## When to use

Use when magnitudes fed into a regression (e.g. photometric redshift GLM) are correlated and you need uncorrelated explanatory variables computed robustly on combined training+test data.

## When NOT to use

- When the covariates are already uncorrelated or physically independent inputs — PCA on the complete set would add unnecessary preprocessing and mixing between training and test samples.
- When the downstream model requires interpretable per-band coefficients — PCs of the magnitude set lose direct magnitude-level interpretation.
- When the goal is a simple formulation without interaction terms capturing data complexity — the paper notes such simple tests are inadequate, but if a strictly linear, band-wise model is required, PCs may not be the right explanatory variables.

## Inputs

- Photometric catalogue of galaxy magnitudes (training and test sets combined), e.g. PHAT0 or SDSS DR10 magnitudes as a pandas/numpy DataFrame or R data frame

## Outputs

- PCA basis / decomposition object (loadings sufficient to project test data)
- PC score matrix for training and test galaxies (e.g. PC1..PC6 columns)
- Cumulative explained-variance diagnostic supporting the ~99.5% threshold

## How to apply

Run robust principal component analysis (Candes et al. 2011; de Souza et al. 2014c) on the COMPLETE data set — training and test together — so the PCs are not dominated by either sample. In the PHAT0 application, the threshold on cumulative percentage of total variance was set to ~99.5% to determine the number of PCs; at least 6 components were required for PHAT0 to reach 99.5%. Project both training and test magnitudes onto the stored PCA basis to obtain PC scores, then pass these PCs as the covariates of the downstream gamma GLM (e.g. factorial formula zphot ~ PC1 + ... + interactions). Note that R and Python implementations solve the SVD matrices slightly differently, so small cross-implementation differences are expected.

## Related tools

- **Python** (Implementation language for the robust PCA + GLM pipeline (CosmoPhotoz package, scikit-learn/statsmodels ecosystem)) — https://github.com/COINtoolbox/COSMOPhotoz/
- **R** (Implementation language for the robust PCA + GLM photo-z code (Appendix A of the paper)) — https://github.com/COINtoolbox/COSMOPhotoz/

## Examples

```
from sklearn.decomposition import PCA; pca = PCA(n_components=0.995, svd_solver='full'); X_pc = pca.fit_transform(mags_all)
```

## Evaluation signals

- Cumulative explained variance of retained PCs meets or exceeds the ~99.5% threshold (for PHAT0 this required at least 6 components).
- PCA was fitted on the complete data set (training and test together), not on the training split alone.
- Projected PC scores for the test set are finite and can be fed into the gamma GLM, yielding positive continuous z_phot predictions for every test galaxy.
- Downstream GLM using the PCs (with interaction terms) outperforms a simple formulation without interactions — the paper states the simple formulation is 'inadequate in capturing the complexity of the data'.
- R and Python runs differ only slightly (SVD solving), so cross-checking both implementations should give near-identical retained-variance and component counts.

## Limitations

- Slight differences exist between the R and Python codes in how SVD matrices are solved in the PCA step, so results are not bit-identical across implementations.
- Fitting the PCA on the complete data set (training + test) is a design choice of the method; using it on truly prospective/unseen data requires storing the basis and projecting rather than refitting.
- The PCA is only the preprocessing stage — accuracy claims (catastrophic outlier rates ~1% simulated, ~2% real) depend on the subsequent gamma-family, log-link GLM fit and adequate training-set size (a few thousand galaxies).

## Evidence

- [methods] Robust PCA on complete data set: [evidence span withheld — anchor status: not_found]
- [discussion] R/Python SVD differences: [evidence span withheld — anchor status: not_found]
