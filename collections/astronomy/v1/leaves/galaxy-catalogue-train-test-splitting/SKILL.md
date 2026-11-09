---
name: galaxy-catalogue-train-test-splitting
description: Use when a galaxy photometric catalogue (e.g., PHAT0 magnitudes with
  spectroscopic redshifts) must be partitioned into training/test sets before fitting
  a photo-z model, so outlier rates are evaluated on unseen galaxies.
when_to_use_negative:
- When the goal is to investigate a preference for under/over-estimation of photo-z
  — the authors note this would require a k-fold cross-validation analysis, which
  was out of scope
- When the input is already a fixed train/test pair shipped with the pipeline (e.g.,
  the CosmoPhotoz PHAT0train/PHAT0test files) and no re-partitioning is intended
- When the data are not galaxy magnitudes with spectroscopic redshifts (e.g., images
  or spectra), since the method's PCA-on-magnitudes design assumes tabular photometry
tools:
- name: CosmoPhotoz
  role: Supplies the PHAT0 train/test galaxy catalogues and implements the GLM photo-z
    pipeline in which the split is applied
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
- name: R
  role: Platform in which the random train/test splitting, robust PCA, and gamma-family
    GLM fitting are coded
- name: Python
  role: Alternative implementation platform (with statsmodels) for the split, PCA,
    and GLM photo-z workflow
  repo: github.com/statsmodels
- name: R/Shiny
  role: Web application through which users can upload their data set and run the
    split-and-fit photo-z procedure
  repo: https://cosmostatisticsinitiative.shinyapps.io/Cosmo Photoz
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.002
    title: 'The overlooked potential of Generalized Linear Models in astronomy-II:
      Gamma regression and photometric redshifts'
  source_content_hash: sha256:26af7851bc30f64d2c31a2808d34ca72a1c772f3b2f00fd6bdced0506b36f727
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/COINtoolbox/COSMOPhotoz
  tools:
  - CosmoPhotoz
  - Python
  - R
  - R/Shiny
  - Shiny
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# galaxy-catalogue-train-test-splitting

## Summary

Randomly partitions a galaxy photometric catalogue into disjoint training and test sets (training sample holding at least 10% of galaxies) so that photometric redshift quality metrics (bias, rms of delta-z, catastrophic outlier rate) are measured on truly held-out objects.

## When to use

Use when a galaxy photometric catalogue (e.g., PHAT0 magnitudes with spectroscopic redshifts) must be partitioned into training/test sets before fitting a photo-z model, so outlier rates are evaluated on unseen galaxies.

## When NOT to use

- When the goal is to investigate a preference for under/over-estimation of photo-z — the authors note this would require a k-fold cross-validation analysis, which was out of scope
- When the input is already a fixed train/test pair shipped with the pipeline (e.g., the CosmoPhotoz PHAT0train/PHAT0test files) and no re-partitioning is intended
- When the data are not galaxy magnitudes with spectroscopic redshifts (e.g., images or spectra), since the method's PCA-on-magnitudes design assumes tabular photometry

## Inputs

- Galaxy photometric catalogue with observed magnitudes and spectroscopic redshifts (e.g., PHAT0 simulated catalogue from the CosmoPhotoz package; PHAT0train: 161,042 galaxies, PHAT0test: 8,478 galaxies)
- Random split specification: training fraction of at least 10% (paper uses 1:9 train:test)

## Outputs

- Disjoint training and test index sets / catalogue subsets
- Test-set photo-z predictions with quality metrics: bias (delta-z mean), rms(delta-z), catastrophic outlier fraction (|z_phot - z_spec|/(1 + z_spec) > 0.10), e.g. written to phat0_glm_metrics.csv

## How to apply

Randomly split the galaxy catalogue into training and test sets with the training sample holding at least 10% of the number of galaxies; for the PHAT0 simulated catalogue the authors used a 1:9 ratio, yielding 33,904 training and 135,616 testing galaxies. Crucially, robust principal component analysis is carried out on the complete data set (training and test together) before fitting, so the split defines only which objects train the gamma-family GLM with log link and which objects are predicted. Fit the GLM via iteratively reweighted least squares on the training PCs (cumulative variance threshold ~99.5%, i.e. 6 PCs for PHAT0), then predict photo-z for the test set using its PC projections and the best-fit GLM. Judge the split correct if the test set is never used in fitting and the resulting metrics (bias ~0.033, rms(delta-z) ~0.025, catastrophic outlier rate ~1.4% for PHAT0) reproduce on the held-out objects.

## Related tools

- **CosmoPhotoz** (Supplies the PHAT0 train/test galaxy catalogues and implements the GLM photo-z pipeline in which the split is applied) — https://github.com/COINtoolbox/COSMOPhotoz/
- **R** (Platform in which the random train/test splitting, robust PCA, and gamma-family GLM fitting are coded)
- **Python** (Alternative implementation platform (with statsmodels) for the split, PCA, and GLM photo-z workflow) — github.com/statsmodels
- **R/Shiny** (Web application through which users can upload their data set and run the split-and-fit photo-z procedure) — https://cosmostatisticsinitiative.shinyapps.io/Cosmo Photoz

## Examples

```
Rscript -e 'set.seed(1); idx <- sample(seq_len(nrow(phat0)), size = 0.1 * nrow(phat0)); train <- phat0[idx, ]; test <- phat0[-idx, ]'
```

## Evaluation signals

- The training sample holds at least 10% of the galaxies; a 1:9 split of PHAT0 must yield exactly 33,904 training and 135,616 testing galaxies
- Training and test index sets are disjoint — no galaxy appears in both subsets
- Test-set predictions use only the test-set PC projections and the training-fit GLM, so held-out metrics are uncontaminated by fitting
- Recomputed test metrics fall within tolerance of the reported PHAT0 values: bias ~0.033, rms(delta-z) ~0.025, catastrophic outlier rate ~1.4%
- The output CSV contains the three named metric fields (bias, rms, catastrophic outlier rate)

## Limitations

- Investigating under/over-estimation of photo-z within the GLM framework would require a k-fold cross-validation analysis, which was out of scope; a single random split cannot probe this
- Slight differences exist between the R and Python implementations in how SVD matrices are solved in the PCA step and how the GLM is minimised, so split-dependent results may differ slightly between platforms
- Default gamma family and inverse power link may not be fully optimal in all situations, so split results are model-family dependent

## Evidence

- [methods] PCA runs on complete data set before fitting: [evidence span withheld — anchor status: not_found]
- [results] k-fold cross-validation out of scope: [evidence span withheld — anchor status: not_found]
