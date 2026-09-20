---
name: redshift-residual-binned-scatter-analysis
description: Use when you have predicted photo-zs and spectroscopic redshifts from
  a gamma-family GLM (log link) fit on PCs of galaxy magnitudes and need to verify
  non-negativity and redshift-dependent residual scatter.
when_to_use_negative:
- You only have the fitted GLM model or PCs but no paired predicted photo-z and spectroscopic
  redshift values for a test sample — there are no residuals to bin
- Your predictions come from a non-gamma model (e.g. Gaussian noise assumption); the
  non-negativity and mean-dependent variance invariants being tested are specific
  to the gamma family with log link
- You need to investigate systematic under/over-estimation of photo-z; the paper notes
  this would require a k-fold cross-validation analysis, which was out of scope here
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0091
tools:
- name: Python
  role: Compute per-bin residual scatter, check non-negativity, and render density
    scatter/violin plots (matplotlib/seaborn)
- name: statsmodels
  role: Fit the gamma-family GLM with log link whose test-set predictions feed the
    residual scatter analysis
  repo: github.com/statsmodels
- name: seaborn
  role: Statistical visualization of z_phot vs z_spec density scatter and residual
    violin plots
  repo: github.com/mwaskom/seaborn
- name: R
  role: 'Alternative implementation: fit gamma GLM via glm() and produce ggplot2 diagnostics
    of binned residual scatter'
- name: CosmoPhotoz
  role: Reference package that produces the GLM photo-z estimates being diagnosed;
    available in R, R/Shiny, and Python
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.002
    title: 'The overlooked potential of Generalized Linear Models in astronomy-II:
      Gamma regression and photometric redshifts'
  source_content_hash: sha256:c2872c48796139ae33b454cc775895773575765b813da264f6435e50ae6d1e95
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/github.com/statsmodels
  tools:
  - CosmoPhotoz
  - Python
  - R
  - seaborn
  - statsmodels
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# Redshift residual binned scatter analysis

## Summary

A diagnostic skill for gamma-family GLM photometric redshift predictions that verifies the two inherent gamma features claimed for extrapolated predictions — non-negativity of predicted photo-zs and heteroscedastic, redshift-dependent scatter — by binning residuals by spectroscopic redshift and reporting catastrophic outlier rates.

## When to use

Use when you have predicted photo-zs and spectroscopic redshifts from a gamma-family GLM (log link) fit on PCs of galaxy magnitudes and need to verify non-negativity and redshift-dependent residual scatter.

## When NOT to use

- You only have the fitted GLM model or PCs but no paired predicted photo-z and spectroscopic redshift values for a test sample — there are no residuals to bin
- Your predictions come from a non-gamma model (e.g. Gaussian noise assumption); the non-negativity and mean-dependent variance invariants being tested are specific to the gamma family with log link
- You need to investigate systematic under/over-estimation of photo-z; the paper notes this would require a k-fold cross-validation analysis, which was out of scope here

## Inputs

- Predicted photo-z values for the test sample from a gamma-family GLM with log link
- Spectroscopic redshifts (z_spec) for the same test galaxies
- Principal components from robust PCA on the complete data set (training + test), retaining ~99.5% cumulative variance

## Outputs

- Count of negative predicted redshifts (expected: zero) and continuity check
- Per-redshift-bin residual scatter (std of z_phot - z_spec and of |z_phot - z_spec|/(1+z_spec))
- 2D density scatter plot of z_phot vs z_spec and violin plot of residuals across redshift bins (e.g. gamma_glm_diagnostics.png)
- Diagnostic report (e.g. gamma_glm_diagnostic_report.md) with non-negativity counts, scatter trend, and catastrophic outlier rates under O(a) and O(b)

## How to apply

Load the predicted photo-zs and spectroscopic redshifts for the test sample produced by the gamma-family GLM fit with log link, using PCs from robust PCA at a ~99.5% cumulative-variance threshold. First scan all predictions for negative entries and confirm the predictions form a continuous distribution, since the gamma family with log link guarantees positive means. Then bin the test sample by spectroscopic redshift and compute per-bin residual scatter, e.g. the standard deviation of z_phot - z_spec and of |z_phot - z_spec|/(1+z_spec); heteroscedasticity is supported when the per-bin scatter increases monotonically with redshift, consistent with the gamma family's variance increasing with the mean. Render a 2D density scatter of z_phot vs z_spec plus a violin plot of residuals across redshift bins, and compute catastrophic outlier rates under both definitions O(a): |z_phot - z_spec| > 0.10 and O(b): |z_phot - z_spec|/(1+z_spec) > 0.10, with the paper's ~1% (simulated) and ~2% (real data) levels as reference context. Summarize the non-negativity counts and scatter trend in a written diagnostic report.

## Related tools

- **Python** (Compute per-bin residual scatter, check non-negativity, and render density scatter/violin plots (matplotlib/seaborn))
- **statsmodels** (Fit the gamma-family GLM with log link whose test-set predictions feed the residual scatter analysis) — github.com/statsmodels
- **seaborn** (Statistical visualization of z_phot vs z_spec density scatter and residual violin plots) — github.com/mwaskom/seaborn
- **R** (Alternative implementation: fit gamma GLM via glm() and produce ggplot2 diagnostics of binned residual scatter)
- **CosmoPhotoz** (Reference package that produces the GLM photo-z estimates being diagnosed; available in R, R/Shiny, and Python) — https://github.com/COINtoolbox/COSMOPhotoz/

## Examples

```
import statsmodels.api as sm; fit = sm.GLM(z_spec_train, X_train, family=sm.families.Gamma(sm.families.links.Log())).fit(); z_phot = fit.predict(X_test)
```

## Evaluation signals

- The report records zero negative predicted redshifts, as required by the gamma family with log link
- Per-bin residual scatter increases monotonically with spectroscopic redshift, consistent with the gamma variance growing with the mean
- Catastrophic outlier rates under O(a): |z_phot - z_spec| > 0.10 and O(b): |z_phot - z_spec|/(1+z_spec) > 0.10 are reported and comparable to the paper's ~1% (simulated) and ~2% (real data) levels
- The PCA step retained at least ~99.5% cumulative variance before the GLM was fit (e.g. 6 components for PHAT0)
- Both plots (density scatter of z_phot vs z_spec and residual violin plot) and the markdown diagnostic report are actually generated on disk

## Limitations

- Default gamma family and inverse power link may not be fully optimal in all situations when the user does not interfere
- Slight differences exist between the R and Python implementations in how they solve the SVD matrices in the PCA step and minimize the GLM, so results are not fully reconciled across the two
- Investigating preference for under/over-estimation of photo-z would require a k-fold cross-validation analysis, which the paper explicitly declared out of scope

## Evidence

- [methods] The predicted photo-z for the test data was calculated using the principal component projections of the test data set and the best-fit GLM: [evidence span withheld — anchor status: not_found]
- [methods] we employed a metric commonly used in the literature, specifically, the catastrophic error (or outlier rate/fraction): [evidence span withheld — anchor status: not_found]
