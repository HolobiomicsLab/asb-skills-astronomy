---
name: gamma-glm-photoz-regression
description: Use when predicting galaxy photo-z from broadband magnitudes (e.g. PHAT0/PHAT1/SDSS
  DR10 catalogues) with a fast, statistically principled regression whose predictions
  must remain non-negative and redshift-dependent in scatter.
when_to_use_negative:
- A simple additive GLM formula without interaction terms is being considered — the
  paper notes such tests are 'inadequate in capturing the complexity of the data'
- The analysis requires assessing systematic under/over-estimation of photo-z — the
  paper states this would require a k-fold cross-validation analysis which is out
  of scope
- A fully optimal link/family is required without user tuning — the package defaults
  to the gamma family and inverse power link, 'which may not be fully optimal in all
  situations'
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: R
  role: Fits the gamma-family GLM with log link via the factorial formula with interaction
    terms using iteratively reweighted least squares (codes in Appendix A)
- name: Python
  role: Alternative implementation of the PCA + GLM photo-z pipeline (codes in Appendix
    B); statsmodels can fit the GLM
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
- name: Shiny
  role: Web application where users can upload their data set and run the photo-z
    estimation without local installation
  repo: https://cosmostatisticsinitiative.shinyapps.io/Cosmo Photoz
- name: CosmoPhotoz
  role: User-friendly package that determines photometric redshifts from galaxy magnitudes
    using GLMs which reproduce the physical aspects of the output distribution
  repo: https://github.com/COINtoolbox/CosmoPhotoz
- name: seaborn
  role: Statistical data visualization for diagnostic plots (e.g. 2D density scatter
    of z_phot vs z_spec, residual violin plots)
  repo: https://github.com/mwaskom/seaborn
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.002
    title: 'The overlooked potential of Generalized Linear Models in astronomy-II:
      Gamma regression and photometric redshifts'
  source_content_hash: sha256:bedf7633d0eca3a9fb82e047e87b586f6537a6539dfd77142f9df3b626ea0e22
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/COINtoolbox/COSMOPhotoz
  tools:
  - CosmoPhotoz
  - Python
  - R
  - Shiny
  - seaborn
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# gamma-glm-photoz-regression

## Summary

Estimate galaxy photometric redshifts by fitting a gamma-family GLM with a log link to principal components of observed magnitudes, using interaction terms and iteratively reweighted least squares. The gamma family preserves physically meaningful prediction features (non-negativity and heteroscedastic scatter) and achieves ~1% (simulated) to ~2% (real data) catastrophic outlier rates within seconds on a desktop computer.

## When to use

Use when predicting galaxy photo-z from broadband magnitudes (e.g. PHAT0/PHAT1/SDSS DR10 catalogues) with a fast, statistically principled regression whose predictions must remain non-negative and redshift-dependent in scatter.

## When NOT to use

- A simple additive GLM formula without interaction terms is being considered — the paper notes such tests are 'inadequate in capturing the complexity of the data'
- The analysis requires assessing systematic under/over-estimation of photo-z — the paper states this would require a k-fold cross-validation analysis which is out of scope
- A fully optimal link/family is required without user tuning — the package defaults to the gamma family and inverse power link, 'which may not be fully optimal in all situations'

## Inputs

- Photometric catalogue of galaxies with observed magnitudes and spectroscopic redshifts (e.g. PHAT0 simulated catalogue, PHAT1, or a subset of SDSS Data Release 10)
- Train/test split specification (training sample at least 10% of galaxies; paper used 1:9 ratio)
- Number of principal components to use as explanatory variables (6, 8, or 10 in the paper's sweep)

## Outputs

- Best-fit gamma-family GLM (log link) over PC interaction terms
- Predicted photo-z values for the test galaxies
- Catastrophic outlier rate O(a) = |z_phot - z_spec| > 0.10 (and optionally O(b) = |z_phot - z_spec|/(1+z_spec) > 0.10)
- Wall-clock fitting time per PC configuration (e.g. 170 s / 1200 s / 4948 s for 6/8/10 PCs)

## How to apply

Randomly split the photometric catalogue into training and test sets (training holds at least 10% of galaxies; the paper used 1:9, giving 33,904 training and 135,616 test PHAT0 galaxies). Run robust PCA on the complete data set (training + test) and select the number of PCs using a cumulative variance threshold of ~99.5% (at least 6 components for PHAT0); using more PCs trades runtime for accuracy (6 PCs: 4.4% outliers in 170 s; 8 PCs: 2.5% in 1200 s; 10 PCs: 1.4% in 4948 s). Fit a gamma-family GLM with a log link using the factorial R formula with crossing/interaction terms (e.g. PC1*PC2*...*PCn), fitted by iteratively reweighted least squares — simple additive formulations without interaction terms are inadequate for capturing the data complexity. Predict photo-z for the test set using the PC projections and the best-fit GLM, then evaluate with the catastrophic outlier rate O(a) (|z_phot - z_spec| > 0.10) and optionally O(b) (|z_phot - z_spec|/(1+z_spec) > 0.10). The CosmoPhotoz package implements this in R, Python, and an R/Shiny web application.

## Related tools

- **R** (Fits the gamma-family GLM with log link via the factorial formula with interaction terms using iteratively reweighted least squares (codes in Appendix A))
- **Python** (Alternative implementation of the PCA + GLM photo-z pipeline (codes in Appendix B); statsmodels can fit the GLM) — https://github.com/COINtoolbox/COSMOPhotoz/
- **Shiny** (Web application where users can upload their data set and run the photo-z estimation without local installation) — https://cosmostatisticsinitiative.shinyapps.io/Cosmo Photoz
- **CosmoPhotoz** (User-friendly package that determines photometric redshifts from galaxy magnitudes using GLMs which reproduce the physical aspects of the output distribution) — https://github.com/COINtoolbox/CosmoPhotoz
- **seaborn** (Statistical data visualization for diagnostic plots (e.g. 2D density scatter of z_phot vs z_spec, residual violin plots)) — https://github.com/mwaskom/seaborn

## Examples

```
glm(z_spec ~ PC1*PC2*PC3*PC4*PC5*PC6, family = Gamma(link = "log"), data = train_pcs)
```

## Evaluation signals

- Reproduced PHAT0 trade-off table shows catastrophic outlier rates of 4.4% (6 PCs), 2.5% (8 PCs) and 1.4% (10 PCs) with fitting times of approximately 170 s, 1200 s and 4948 s
- Zero negative predicted redshifts when scanning all predicted values (gamma family guarantees non-negativity)
- Per-redshift-bin residual scatter increases monotonically with redshift, consistent with the gamma family's variance increasing with the mean
- PCA decomposition retains at least ~99.5% cumulative variance (at least 6 components for PHAT0)
- Catastrophic outlier rates fall near the paper's reported ~1% (simulated) and ~2% (real data) reference levels

## Limitations

- Slight differences exist between the R and Python implementations in how they solve the SVD matrices in the PCA step and how they minimise the GLM
- Reported performance is tied to a specific low-end hardware/OS configuration (AMD Athlon X2 Dual-Core QL-64, 1.7 GB RAM, Ubuntu 10.04), so runtimes may differ elsewhere
- If the user does not interfere, the gamma family and inverse power link are used, which may not be fully optimal in all situations
- Investigating under/over-estimation of photo-z would require a k-fold cross-validation analysis, which is out of scope
- The web-based (Shiny) interface requires no local installation but actual processing may be slower

## Evidence
