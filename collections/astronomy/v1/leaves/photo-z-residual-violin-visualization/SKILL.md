---
name: photo-z-residual-violin-visualization
description: Use when you need to visually verify that predicted photo-zs from a fitted
  gamma-family GLM (log link) are non-negative, continuous, and show redshift-dependent
  residual scatter before reporting quality metrics.
when_to_use_negative:
- When only aggregate quality metrics (e.g., catastrophic outlier rate) are needed
  and no distributional diagnostic of residuals is required
- When the photo-z model is not a gamma-family GLM with log link, so non-negativity
  and mean-linked variance are not inherent model properties to verify
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: seaborn
  role: Python statistical visualization library used to draw the residual violin
    plot and density scatter of photo-z diagnostics
  repo: https://github.com/mwaskom/seaborn
- name: matplotlib
  role: Python plotting backend underlying the diagnostic figure (gamma_glm_diagnostics.png)
  repo: https://github.com/mwaskom/seaborn
- name: R/ggplot2
  role: Alternative R path to render the density scatter and violin plot of residuals
    across redshift bins
- name: CosmoPhotoz
  role: R/Python/Shiny package that produces the gamma-GLM photo-z predictions being
    visualized
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.002
    title: 'The overlooked potential of Generalized Linear Models in astronomy-II:
      Gamma regression and photometric redshifts'
  source_content_hash: sha256:dd13a14ac1717aa8f634fde986f57e529393c8aa2c1b0e7d0ca22312b93b8ef2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/mwaskom/seaborn
  tools:
  - CosmoPhotoz
  - Python
  - R
  - R/ggplot2
  - matplotlib
  - seaborn
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# photo-z-residual-violin-visualization

## Summary

Visual diagnostic for gamma-GLM photometric redshifts: render a 2D density scatter of z_phot vs z_spec plus a violin plot of residuals across redshift bins, exposing non-negativity, continuity, and heteroscedastic (redshift-dependent) scatter of predictions.

## When to use

Use when you need to visually verify that predicted photo-zs from a fitted gamma-family GLM (log link) are non-negative, continuous, and show redshift-dependent residual scatter before reporting quality metrics.

## When NOT to use

- When only aggregate quality metrics (e.g., catastrophic outlier rate) are needed and no distributional diagnostic of residuals is required
- When the photo-z model is not a gamma-family GLM with log link, so non-negativity and mean-linked variance are not inherent model properties to verify

## Inputs

- Predicted photo-z values for the test sample from the best-fit gamma-family GLM (log link)
- Spectroscopic redshifts for the same test galaxies (e.g., from PHAT0 simulation, PHAT1, or SDSS DR10 samples)
- Redshift bin edges defining test-sample bins by z_spec

## Outputs

- gamma_glm_diagnostics.png containing a 2D density scatter of z_phot vs z_spec and a violin plot of residuals across redshift bins
- gamma_glm_diagnostic_report.md summarizing non-negativity check counts, per-bin scatter trend, and outlier rates under O(a) and O(b) definitions

## How to apply

Load the predicted photo-zs and spectroscopic redshifts from the gamma-GLM test sample (built on robust-PCA PCs retained at a ~99.5% cumulative-variance threshold). First scan predictions for negative entries and confirm the distribution is continuous in R or Python (statsmodels/R glm), since the gamma family with log link should guarantee strictly positive means. Bin the test sample by spectroscopic redshift and compute per-bin residual scatter (std of z_phot - z_spec, and of |z_phot - z_spec|/(1+z_spec)); heteroscedasticity is supported if per-bin scatter increases monotonically with redshift, consistent with the gamma variance rising with the mean. Render a 2D density scatter of z_phot vs z_spec and a violin plot of residuals across redshift bins with matplotlib/seaborn or ggplot2, saving gamma_glm_diagnostics.png. Write gamma_glm_diagnostic_report.md recording the non-negativity counts, scatter trend, and catastrophic outlier rates under O(a): |z_phot - z_spec| > 0.10 and O(b): |z_phot - z_spec|/(1+z_spec) > 0.10.

## Related tools

- **seaborn** (Python statistical visualization library used to draw the residual violin plot and density scatter of photo-z diagnostics) — https://github.com/mwaskom/seaborn
- **matplotlib** (Python plotting backend underlying the diagnostic figure (gamma_glm_diagnostics.png)) — https://github.com/mwaskom/seaborn
- **R/ggplot2** (Alternative R path to render the density scatter and violin plot of residuals across redshift bins)
- **CosmoPhotoz** (R/Python/Shiny package that produces the gamma-GLM photo-z predictions being visualized) — https://github.com/COINtoolbox/COSMOPhotoz/

## Examples

```
import seaborn as sns, matplotlib.pyplot as plt; sns.violinplot(x='z_bin', y='resid', data=df); plt.savefig('gamma_glm_diagnostics.png')
```

## Evaluation signals

- The diagnostic report records zero negative predicted redshifts across the test sample
- Per-bin residual scatter (std of z_phot - z_spec and of |z_phot - z_spec|/(1+z_spec)) increases monotonically with redshift
- Predicted photo-zs form a continuous distribution in R or Python with no gaps or invalid values
- Catastrophic outlier rates under O(a) (|z_phot - z_spec| > 0.10) and O(b) (|z_phot - z_spec|/(1+z_spec) > 0.10) are reported, with ~1% (simulated) and ~2% (real data) as reference levels
- gamma_glm_diagnostics.png renders both the density scatter and residual violin plot without plotting errors

## Limitations

- A k-fold cross-validation to assess under/over-estimation of photo-z was explicitly out of scope in the paper, so the violin plot alone cannot establish estimation bias
- Slight differences between the R and Python implementations in SVD solving (PCA) and GLM minimisation are not fully reconciled, so residuals may differ slightly between the two toolchains
- For visual clarity one outlier was clipped at y = 6.5 in the paper's Q-Q plot; similar clipping choices in diagnostic figures can hide extreme residuals

## Evidence

- [results] The paper asserts that GLM photo-z predictions retain two inherent gamma features: [evidence span withheld — anchor status: not_found]
- [methods] Outlier rate definitions provide the quantitative context reported alongside the visual diagnostics: [evidence span withheld — anchor status: not_found]
