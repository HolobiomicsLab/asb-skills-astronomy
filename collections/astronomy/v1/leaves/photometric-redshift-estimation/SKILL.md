---
name: photometric-redshift-estimation
description: Use when you have a galaxy catalogue of u'g'r'i'z' (or similar multi-band)
  magnitudes plus spectroscopic redshifts and need fast, reliable photo-z predictions
  with gamma-distributed, non-negative output.
when_to_use_negative:
- The analysis goal is investigating under/over-estimation bias of photo-z, which
  would require a k-fold cross-validation analysis that the paper explicitly left
  out of scope
- You need a custom link function — the package defaults to the gamma family with
  inverse power link when the user does not intervene, which may not be fully optimal
  in all situations
- You require exact numerical agreement between R and Python runs, since they solve
  the SVD matrices in the PCA step and minimise the GLM slightly differently
edam_operation: http://edamontology.org/operation_0320
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Fit the gamma-family, log-link GLM with the factorial formula via iteratively
    re-weighted least squares (glm); code provided in Appendix A
- name: Python
  role: Alternative implementation of the PCA + gamma GLM pipeline (Appendix B), e.g.
    via statsmodels GLM; package released in Python
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
- name: R/Shiny
  role: Interactive web application (Shiny platform) where users upload their data
    set to run the photo-z GLM fit without local installation
  repo: https://cosmostatisticsinitiative.shinyapps.io/Cosmo Photoz
- name: CosmoPhotoz
  role: The public user-friendly package (R, R/Shiny webapp, Python) that determines
    photometric redshifts from galaxy magnitudes using GLMs
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
- name: statsmodels
  role: Python library for fitting the gamma-family GLM by IRLS in the pipeline
  repo: github.com/statsmodels
- name: seaborn
  role: Statistical data visualization used for rendering photo-z diagnostic plots
    (e.g., density scatter of z_phot vs z_spec)
  repo: https://github.com/mwaskom/seaborn
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_007
  - task_006
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.002
    title: 'The overlooked potential of Generalized Linear Models in astronomy-II:
      Gamma regression and photometric redshifts'
  source_content_hash: sha256:1ca26125ca310620ff9a75e2280a89a5abc9c1ebdb0cb3cc2ecd82e91017b81f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/COINtoolbox/COSMOPhotoz
  tools:
  - CosmoPhotoz
  - Python
  - R
  - R/Shiny
  - seaborn
  - statsmodels
  edam_operation: http://edamontology.org/operation_0320
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# photometric-redshift-estimation

## Summary

Estimate galaxy photometric redshifts by applying robust PCA to observed magnitudes (cumulative variance threshold ~99.5%) and fitting a gamma-family, log-link GLM with factorial interaction terms via iteratively re-weighted least squares, evaluating catastrophic outlier rates O(a)/O(b).

## When to use

Use when you have a galaxy catalogue of u'g'r'i'z' (or similar multi-band) magnitudes plus spectroscopic redshifts and need fast, reliable photo-z predictions with gamma-distributed, non-negative output.

## When NOT to use

- The analysis goal is investigating under/over-estimation bias of photo-z, which would require a k-fold cross-validation analysis that the paper explicitly left out of scope
- You need a custom link function — the package defaults to the gamma family with inverse power link when the user does not intervene, which may not be fully optimal in all situations
- You require exact numerical agreement between R and Python runs, since they solve the SVD matrices in the PCA step and minimise the GLM slightly differently

## Inputs

- Galaxy photometric catalogue with multi-band magnitudes (e.g., PHAT0 simulated catalogue; SDSS DR10 sample of 1,347,640 galaxies with u'g'r'i'z' magnitudes and z = 0-1.0)
- Spectroscopic redshifts for the training split
- Cumulative-variance threshold (~99.5%) and number of PCs (n = 4-10)
- SQL query for the SDSS DR10 galaxy sample (Appendix C of the paper)

## Outputs

- photoz_predictions.csv with a per-galaxy positive continuous z_phot for each test-set galaxy
- Fitted GLM model object (fitted_glm.pkl or RDS)
- Catastrophic outlier rates O(a) and O(b) (e.g., sdss_photoz_metrics.csv)
- PC projections / stored robust-PCA basis

## How to apply

Split the catalogue randomly into training and test sets with the training sample holding at least 10% of galaxies (e.g., PHAT0 used a 1:9 ratio giving 33,904 training and 135,616 test galaxies). Run robust principal component analysis (Candes et al. 2011-style) on the COMPLETE data set — training plus test — and select the number of PCs with a cumulative total-variance threshold of ~99.5% (PHAT0 needed at least 6 components). Fit a gamma-family GLM with a log link using the factorial formula z_phot ~ PC1*PC2*...*PCn + C (the crossing operator includes interaction terms, since simple formulations without interactions are inadequate for the data's complexity), fitted by iteratively re-weighted least squares. Predict photo-z for the test data from the PC projections and the best-fit GLM, then quantify quality with the catastrophic error O(a) = |z_phot - z_spec| > 0.10 (and optionally O(b) = |z_phot - z_spec|/(1+z_spec) > 0.10 for dereddened magnitudes). Benchmarks: SDSS DR10 gamma GLM with 4 PCs gives O(a) = 8.087% and with 5 PCs O(a) = 6.439%; PHAT0 with 6/8/10 PCs gives 4.4%/2.5%/1.4% in ~170 s/1200 s/4948 s.

## Related tools

- **R** (Fit the gamma-family, log-link GLM with the factorial formula via iteratively re-weighted least squares (glm); code provided in Appendix A)
- **Python** (Alternative implementation of the PCA + gamma GLM pipeline (Appendix B), e.g. via statsmodels GLM; package released in Python) — https://github.com/COINtoolbox/COSMOPhotoz/
- **R/Shiny** (Interactive web application (Shiny platform) where users upload their data set to run the photo-z GLM fit without local installation) — https://cosmostatisticsinitiative.shinyapps.io/Cosmo Photoz
- **CosmoPhotoz** (The public user-friendly package (R, R/Shiny webapp, Python) that determines photometric redshifts from galaxy magnitudes using GLMs) — https://github.com/COINtoolbox/COSMOPhotoz/
- **statsmodels** (Python library for fitting the gamma-family GLM by IRLS in the pipeline) — github.com/statsmodels
- **seaborn** (Statistical data visualization used for rendering photo-z diagnostic plots (e.g., density scatter of z_phot vs z_spec)) — https://github.com/mwaskom/seaborn

## Examples

```
import statsmodels.api as sm; glm = sm.GLM(z_train, X_train_pcs, family=sm.families.Gamma(sm.families.links.Log())).fit()
```

## Evaluation signals

- Reproduced catastrophic outlier rates match the paper: PHAT0 ~1% (simulated) and SDSS ~2% (real data) as reference levels; SDSS O(a) = 8.087% (4 PCs) and 6.439% (5 PCs)
- PHAT0 PC sweep reproduces the trade-off table: O(a) = 4.4% in ~170 s (6 PCs), 2.5% in ~1200 s (8 PCs), 1.4% in ~4948 s (10 PCs)
- PCA on the complete data set retains at least ~99.5% cumulative variance (PHAT0 required at least 6 components)
- All predicted photo-zs are non-negative and form a continuous distribution, consistent with the gamma family's positive support
- Results are obtained within seconds on a single core with training sets of merely thousands of galaxies

## Limitations

- Slight differences between the R and Python implementations in how they solve the SVD matrices in the PCA step and how they minimise the GLM are not fully reconciled
- Default gamma family and inverse power link may not be fully optimal in all situations if the user does not interfere
- Reported performance is tied to a specific low-end hardware/OS configuration (AMD Athlon X2 Dual-Core QL-64 processor with 1.7 GB RAM on the Ubuntu 10.04 OS)
- K-fold cross-validation for under/over-estimation of photo-z was out of scope; one Q-Q plot outlier was clipped at y = 6.5 for visual reasons

## Evidence

- [methods] PCA on full data + ~99.5% variance threshold: [evidence span withheld — anchor status: not_found]
- [methods] IRLS fitting of the factorial GLM: [evidence span withheld — anchor status: not_found]
- [results] gamma family with log link on PHAT0: [evidence span withheld — anchor status: not_found]
