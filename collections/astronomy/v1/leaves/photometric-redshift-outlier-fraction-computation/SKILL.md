---
name: photometric-redshift-outlier-fraction-computation
description: Use when comparing photo-z method performance via catastrophic outlier
  rates — e.g. tabulating GLM vs baseline codes (ANNz, BPZ, Le PHARE, TPZ, BMA) on
  PHAT0 or SDSS test sets under fixed Δz > 0.10 definitions.
when_to_use_negative:
- Estimating general photo-z accuracy (bias/rms) rather than catastrophic outlier
  fractions — a k-fold cross-validation analysis of under/over-estimation is explicitly
  out of scope in this work
- Applying to raw magnitude catalogues without a fitted photo-z model — the outlier
  fraction requires predicted zphot values from a trained estimator (e.g. the gamma-family
  GLM with log link)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Python
  role: Used to assemble the merged comparison table (pandas) of GLM and baseline-code
    photo-z metrics and save photoz_metrics_comparison.csv
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
- name: CosmoPhotoz
  role: Package producing the GLM photo-z predictions whose catastrophic outlier rates
    are computed; available in Python, R and R/Shiny
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.002
    title: 'The overlooked potential of Generalized Linear Models in astronomy-II:
      Gamma regression and photometric redshifts'
  source_content_hash: sha256:c2bbdcb807cecb3caed05d7cfa941249e647b9f37b4ab28c652312f2fef1cdd2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/COINtoolbox/COSMOPhotoz
  tools:
  - CosmoPhotoz
  - Python
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# photometric-redshift-outlier-fraction-computation

## Summary

Computes and tabulates catastrophic error (outlier) rates for photometric redshift estimates, as used in the gamma-GLM photo-z paper to benchmark GLM results on PHAT0 and SDSS against published codes (Le PHARE, BPZ, ANNz, TPZ, BMA).

## When to use

Use when comparing photo-z method performance via catastrophic outlier rates — e.g. tabulating GLM vs baseline codes (ANNz, BPZ, Le PHARE, TPZ, BMA) on PHAT0 or SDSS test sets under fixed Δz > 0.10 definitions.

## When NOT to use

- Estimating general photo-z accuracy (bias/rms) rather than catastrophic outlier fractions — a k-fold cross-validation analysis of under/over-estimation is explicitly out of scope in this work
- Applying to raw magnitude catalogues without a fitted photo-z model — the outlier fraction requires predicted zphot values from a trained estimator (e.g. the gamma-family GLM with log link)

## Inputs

- Table of predicted photometric redshifts and spectroscopic redshifts for the test set (e.g. PHAT0 test sample of 135,616 galaxies, or SDSS DR10 subset)
- Tabulated bias/rms/outlier values for baseline photo-z codes (ANNz, BPZ, Le PHARE, EAZY, TPZ, BMA) as reported in the article's results tables

## Outputs

- Per-method catastrophic outlier fractions under O(a) (|zphot − zspec| > 0.10) and O(b) (|zphot − zspec|/(1+zspec) > 0.10)
- Merged comparison table photoz_metrics_comparison.csv with columns sample (PHAT0/SDSS), method (GLM vs baseline codes), and reported metrics

## How to apply

After predicting photo-z for the held-out test set (e.g. the 135,616-galaxy PHAT0 test sample from a 1:9 split), classify each object as catastrophic using the threshold Δz = |zphot − zspec| > 0.10 (definition O(a)) and/or Δz = |zphot − zspec|/(1+zspec) > 0.10 (definition O(b)); keep whichever definition each source entry reports. Compute the outlier fraction per method and per principal-component configuration (the gamma-family GLM with log link yields, e.g., 1.367% with 10 PCs, 2.511% with 8 PCs, 4.438% with 6 PCs on PHAT0, and 6.44% with 5 PCs, 8.09% with 4 PCs on SDSS). Assemble a merged pandas table (photoz_metrics_comparison.csv) with columns for sample, method, and both O(a)/O(b) definitions so GLM rates can be placed side-by-side with tabulated baseline-code values. Validate that the recomputed GLM rates are ~1% for simulated (PHAT0) and ~2% for real (SDSS) data at the best configurations, consistent with the article's abstract.

## Related tools

- **Python** (Used to assemble the merged comparison table (pandas) of GLM and baseline-code photo-z metrics and save photoz_metrics_comparison.csv) — https://github.com/COINtoolbox/COSMOPhotoz/
- **CosmoPhotoz** (Package producing the GLM photo-z predictions whose catastrophic outlier rates are computed; available in Python, R and R/Shiny) — https://github.com/COINtoolbox/COSMOPhotoz/

## Examples

```
df = pd.read_csv('photoz_predictions.csv'); out_a = ((df.z_phot - df.z_spec).abs() > 0.10).mean(); out_b = ((df.z_phot - df.z_spec).abs()/(1+df.z_spec) > 0.10).mean()
```

## Evaluation signals

- Recomputed GLM outlier rates reproduce reported values: 1.367% (10 PCs), 2.511% (8 PCs), 4.438% (6 PCs) on PHAT0 and 6.44% (5 PCs), 8.09% (4 PCs) on SDSS
- Best-configuration rates fall at the reported level of ~1% for simulated data and ~2% for real data
- Comparison table contains a row for every tabulated method and sample (GLM plus ANNz, BPZ, Le PHARE, EAZY, TPZ, BMA, etc.)
- Both O(a) and O(b) columns are present in photoz_metrics_comparison.csv, with the article-reported definition retained per entry
- PHAT0 rates at ≥6 PCs (the ~99.5% cumulative-variance threshold) do not exceed ~5% catastrophic errors

## Limitations

- A k-fold cross-validation analysis to investigate under/over-estimation of photo-z was not performed and is out of scope
- Slight differences exist between the R and Python implementations in SVD solving (PCA step) and GLM minimisation, so rates may differ marginally between platforms
- The default gamma family and inverse power link may not be fully optimal in all situations, affecting achievable outlier rates

## Evidence

- [other] catastrophic rates PHAT0/SDSS: [evidence span withheld — anchor status: not_found]
- [methods] outlier definitions: [evidence span withheld — anchor status: not_found]
