---
name: robust-pca-dimensionality-reduction
description: Use when correlated multi-band magnitudes (e.g. PHAT0 u'g'r'i'z' or SDSS
  DR10 photometry) must be compressed into uncorrelated PCs before fitting a gamma
  GLM with log link for photo-z prediction.
when_to_use_negative:
- When the user's simple formulation without interaction terms is intended — the article
  notes such tests are inadequate in capturing the complexity of the data, so PCs
  alone without crossed GLM terms will underperform
- When exact cross-platform reproducibility of PC scores is required — the R and Python
  codes solve the SVD matrices slightly differently and this was not fully reconciled
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3360
tools:
- name: R
  role: Implementation platform for robust PCA and the downstream gamma GLM fit (Appendix
    A code)
  repo: https://github.com/COINtoolbox/CosmoPhotoz
- name: Python
  role: Implementation platform for robust PCA (SVD solving) and GLM minimisation
    (Appendix B code)
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
- name: CosmoPhotoz
  role: Public package providing the PHAT0 train/test catalogues and the R/Python/web
    implementations of the PCA + GLM photo-z method
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_007
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.002
    title: 'The overlooked potential of Generalized Linear Models in astronomy-II:
      Gamma regression and photometric redshifts'
  source_content_hash: sha256:c78e877a3dc22d76d8ea7c34fe912efac0509d56986e713285987bb7068c17f0
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/COINtoolbox/CosmoPhotoz
  tools:
  - CosmoPhotoz
  - Python
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# robust-pca-dimensionality-reduction

## Summary

Robust principal component analysis applied to galaxy photometric magnitudes to produce uncorrelated principal components used as explanatory variables for gamma-family GLM photometric redshift estimation, with the number of PCs selected by a ~99.5% cumulative variance threshold.

## When to use

Use when correlated multi-band magnitudes (e.g. PHAT0 u'g'r'i'z' or SDSS DR10 photometry) must be compressed into uncorrelated PCs before fitting a gamma GLM with log link for photo-z prediction.

## When NOT to use

- When the user's simple formulation without interaction terms is intended — the article notes such tests are inadequate in capturing the complexity of the data, so PCs alone without crossed GLM terms will underperform
- When exact cross-platform reproducibility of PC scores is required — the R and Python codes solve the SVD matrices slightly differently and this was not fully reconciled

## Inputs

- Galaxy photometric catalogue with multi-band magnitudes (e.g. PHAT0 train/test catalogues from the CosmoPhotoz package: PHAT0train 161,042 galaxies, PHAT0test 8,478 galaxies; or SDSS DR10 sample of 1,347,640 galaxies)
- Combined training and test data set for joint robust PCA decomposition
- Cumulative variance threshold parameter (~99.5%) and desired number of PCs (4, 5, 6, 8, or 10)

## Outputs

- PC projections (scores) of the training set for GLM fitting
- PC projections of the test set for photo-z prediction
- Selected number of components and retained cumulative variance (e.g. 6 PCs retaining 99.5% for PHAT0)

## How to apply

Run robust PCA (e.g. following Candès et al. 2011) on the complete data set — training and test sets combined — rather than on the training set alone. Set the threshold on cumulative percentage of total variance to ~99.5% to determine the number of PCs to retain: for PHAT0 this yields at least 6 components, while for SDSS DR10 both 4-PC (99.5% variance) and 5-PC reductions are carried forward to compare catastrophic error rates. Project both training and test magnitudes onto the selected PC basis, then use the PC scores as explanatory variables in the factorial GLM formula zphot ~ PC1 * PC2 * ... * PCn + C (crossing operator, so interaction terms are included). Rationale: simple formulations without interaction terms are inadequate for capturing the data complexity, and PCA removes the strong correlations among observed magnitudes so the GLM receives uncorrelated regressors. Note that R and Python implementations differ slightly in how they solve the SVD matrices in the PCA step, so PC projections may not match exactly across platforms.

## Related tools

- **R** (Implementation platform for robust PCA and the downstream gamma GLM fit (Appendix A code)) — https://github.com/COINtoolbox/CosmoPhotoz
- **Python** (Implementation platform for robust PCA (SVD solving) and GLM minimisation (Appendix B code)) — https://github.com/COINtoolbox/COSMOPhotoz/
- **CosmoPhotoz** (Public package providing the PHAT0 train/test catalogues and the R/Python/web implementations of the PCA + GLM photo-z method) — https://github.com/COINtoolbox/COSMOPhotoz/

## Examples

```
library(CosmoPhotoz); data(PHAT0train, PHAT0test); pcs <- prcomp(rbind(PHAT0train, PHAT0test), robust=TRUE); n_pcs <- which(cumsum(pcs$sdev^2/sum(pcs$sdev^2)) >= 0.995)[1]
```

## Evaluation signals

- The retained components should reach the ~99.5% cumulative variance threshold — for PHAT0 the decomposition must show at least 6 components
- PC projections must be computed for BOTH training and test sets using the same joint decomposition basis
- Downstream photo-z metrics must remain consistent with the published results (e.g. PHAT0 10-PC bias ~0.033, rms(delta-z) ~0.025, outlier rate ~1.4%; PHAT0 6/8/10-PC outlier rates 4.4%/2.5%/1.4%)
- Changing the number of PCs (4 vs 5 on SDSS) should measurably change the catastrophic error rate (8.087% vs 6.439%), confirming the reduction dimensionality was applied as specified

## Limitations

- Slight differences exist between the R and Python implementations in how the SVD matrices are solved in the PCA step and how the GLM is minimised, which were not fully reconciled
- The number of PCs trades accuracy against runtime: adding PCs beyond the 99.5% threshold dramatically increases fitting time (PHAT0: 6 PCs ~170 s vs 10 PCs ~4948 s)
- Performance results are tied to specific hardware; reported fitting times come from a low-end laptop (AMD Athlon X2 QL-64, 1.7 GB RAM)

## Evidence

- [methods] Robust principal component analysis on the complete data set: [evidence span withheld — anchor status: not_found]
- [discussion] R and Python PCA implementations differ in SVD solving: [evidence span withheld — anchor status: not_found]
