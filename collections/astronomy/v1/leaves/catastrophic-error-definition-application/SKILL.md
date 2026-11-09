---
name: catastrophic-error-definition-application
description: Use when predicted photometric redshifts must be scored against spectroscopic
  redshifts using the outlier fraction metric reported in the GLM photo-z article,
  e.g. benchmarking gamma-family GLM fits on PHAT0 or SDSS DR10 against codes like
  Le PHARE, BPZ, ANNz, TPZ, or BMA.
when_to_use_negative:
- The analysis goal is bias/rms scatter characterization rather than outlier-fraction
  benchmarking — the article notes k-fold cross-validation for under/over-estimation
  of photo-z was out of scope
- Inputs are raw magnitudes rather than paired zphot/zspec predictions — the metric
  requires fitted redshift estimates, so run PCA + gamma GLM fitting first
- Comparing studies that use incompatible outlier thresholds (other than the 0.10
  cutoff) without restating definitions, which would invalidate the merged table
tools:
- name: CosmoPhotoz
  role: Reference implementation of the GLM photo-z pipeline whose outputs (predicted
    photo-z for the test set) are scored with the catastrophic-error metric
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
- name: statsmodels
  role: Python library used in the article's workflow for fitting the gamma-family
    GLM whose test-set predictions feed the outlier-rate calculation
  repo: https://github.com/statsmodels
- name: seaborn
  role: Python visualization library from the article's tool stack for plotting photo-z
    diagnostics alongside the computed outlier rates
  repo: https://github.com/mwaskom/seaborn
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.002
    title: 'The overlooked potential of Generalized Linear Models in astronomy-II:
      Gamma regression and photometric redshifts'
  source_content_hash: sha256:46df2dcb2f0a016b998a02ce15c4fb3a0d9286965f7e9dbd523c7c80b54f1785
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/COINtoolbox/COSMOPhotoz
  tools:
  - CosmoPhotoz
  - Python
  - seaborn
  - statsmodels
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# catastrophic-error-definition-application

## Summary

Computes the catastrophic error (outlier rate/fraction) metric used in the article to judge photometric redshift quality, applying the literature-standard definitions O(a) with Δz = |zphot − zspec| > 0.10 and O(b) with Δz = |zphot − zspec|/(1+zspec) > 0.10 to GLM and baseline photo-z predictions.

## When to use

Use when predicted photometric redshifts must be scored against spectroscopic redshifts using the outlier fraction metric reported in the GLM photo-z article, e.g. benchmarking gamma-family GLM fits on PHAT0 or SDSS DR10 against codes like Le PHARE, BPZ, ANNz, TPZ, or BMA.

## When NOT to use

- The analysis goal is bias/rms scatter characterization rather than outlier-fraction benchmarking — the article notes k-fold cross-validation for under/over-estimation of photo-z was out of scope
- Inputs are raw magnitudes rather than paired zphot/zspec predictions — the metric requires fitted redshift estimates, so run PCA + gamma GLM fitting first
- Comparing studies that use incompatible outlier thresholds (other than the 0.10 cutoff) without restating definitions, which would invalidate the merged table

## Inputs

- Predicted photometric redshifts (zphot) for a held-out test set, e.g. from the gamma-family GLM with log link fitted on PCA-reduced magnitudes
- Spectroscopic reference redshifts (zspec) matched to the same objects (PHAT0 simulated catalogue or SDSS DR10 subset)
- Published metric tables for baseline photo-z codes (ANNz, BPZ, Le PHARE, EAZY, TPZ, BMA) with their outlier-definition annotations

## Outputs

- Catastrophic outlier fraction per sample and method under O(a): |zphot − zspec| > 0.10
- Catastrophic outlier fraction per sample and method under O(b): |zphot − zspec|/(1+zspec) > 0.10
- Merged comparison table (e.g. photoz_metrics_comparison.csv) with sample, method, and reported metrics columns

## How to apply

Take the predicted photo-z for the test set (calculated from the principal component projections and the best-fit gamma-family GLM with log link) and pair it with the spectroscopic redshifts. Apply the two literature-standard outlier definitions: O(a) flags objects where Δz = |zphot − zspec| > 0.10, and O(b) flags objects where Δz = |zphot − zspec|/(1+zspec) > 0.10; report each entry under the definition the source article uses. Count flagged objects over the test set to obtain the outlier fraction, which in the article reaches as low as ~1% for simulated data (PHAT0: 1.367% with 10 PCs, 2.511% with 8 PCs, 4.438% with 6 PCs) and ~2% for real data (SDSS: 6.44% with 5 PCs, 8.09% with 4 PCs). Assemble GLM and baseline-code values (ANNz, BPZ, Le PHARE, EAZY, TPZ, BMA) into a merged comparison table (e.g. photoz_metrics_comparison.csv via pandas) with columns for sample, method, and both O(a)/O(b) definitions so cross-code comparison is definition-consistent.

## Related tools

- **CosmoPhotoz** (Reference implementation of the GLM photo-z pipeline whose outputs (predicted photo-z for the test set) are scored with the catastrophic-error metric) — https://github.com/COINtoolbox/COSMOPhotoz/
- **statsmodels** (Python library used in the article's workflow for fitting the gamma-family GLM whose test-set predictions feed the outlier-rate calculation) — https://github.com/statsmodels
- **seaborn** (Python visualization library from the article's tool stack for plotting photo-z diagnostics alongside the computed outlier rates) — https://github.com/mwaskom/seaborn

## Examples

```
import pandas as pd; tbl = pd.DataFrame([{'sample':'PHAT0','method':'GLM','O_b':0.01367},{'sample':'SDSS','method':'GLM','O_b':0.0644}]); tbl.to_csv('photoz_metrics_comparison.csv', index=False)
```

## Evaluation signals

- Recomputed GLM outlier rates match the reported values: ~1.367% (10 PCs), 2.511% (8 PCs), 4.438% (6 PCs) on PHAT0 and 6.44% (5 PCs), 8.09% (4 PCs) on SDSS
- The comparison table contains a row for every tabulated method and sample, with both O(a) and O(b) columns populated
- Each baseline entry is recorded under the same outlier definition the source article reports for that code
- Computed PHAT0 rate is of order ~1% (simulated data) and SDSS rate is of order ~2% baseline expectation (real data) as claimed in the abstract
- Outlier fractions fall in the plausible 1–5% range for the gamma-family GLM configuration (≥6 PCs retaining ~99.5% of variance, 1:9 train/test split)

## Limitations

- The article did not perform k-fold cross-validation, so systematic under/over-estimation of photo-z is not captured by this metric
- Slight differences exist between the R and Python implementations in SVD solving (PCA step) and GLM minimisation, which can shift computed rates slightly
- Reported performance is tied to a specific low-end hardware/OS configuration (AMD Athlon X2 QL-64, 1.7 GB RAM, Ubuntu 10.04); metric values may vary with setup

## Evidence

- [methods] catastrophic-error metric definition: [evidence span withheld — anchor status: not_found]
- [abstract] expected performance range: "We obtain fits that result in catastrophic outlier rates as low as ∼ 1% for simulated and ∼ 2% for real data."
- [methods] prediction step feeding the metric: [evidence span withheld — anchor status: not_found]
- [readme] readme identification of the method: "The method utilises Generalized Linear Models which reproduce the physical aspects of the output distribution."
- [results] PCA component threshold used in the scored configuration: "Decomposition by PCA showed that at least 6 components needed to be used to ensure that 99.5% of the variance was retained"
