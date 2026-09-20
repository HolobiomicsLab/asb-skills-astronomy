---
name: multi-sample-photo-z-comparison-analysis
description: 'Use when benchmarking a photo-z method''s catastrophic outlier rates
  (O(a): |zphot-zspec|>0.10; O(b): |zphot-zspec|/(1+zspec)>0.10) against published
  codes across simulated (PHAT0) and real (SDSS DR10) samples.'
when_to_use_negative:
- The goal is to re-run the GLM photo-z fit itself rather than tabulate/compare already-reported
  metrics — use the CosmoPhotoz fitting workflow instead
- The comparison targets metrics not tabulated in the article (e.g., k-fold cross-validated
  under/over-estimation of photo-z), which the authors explicitly declare out of scope
- Only one sample and one method are involved, so a multi-sample, multi-code merged
  comparison table adds no value
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Python
  role: pandas-based assembly of the merged photoz_metrics_comparison.csv table from
    the article's reported metrics
- name: CosmoPhotoz
  role: source package implementing the gamma-family GLM photo-z estimation whose
    reported catastrophic outlier rates are compared against baseline codes
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
- name: seaborn
  role: statistical visualization library cited by the article for plotting the photo-z
    comparison diagnostics
  repo: https://github.com/mwaskom/seaborn
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.002
    title: 'The overlooked potential of Generalized Linear Models in astronomy-II:
      Gamma regression and photometric redshifts'
  source_content_hash: sha256:950841facbbf149b55e7d477a979f1d8bcfab7f727944b17bb223a52960d53a9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/COINtoolbox/COSMOPhotoz
  tools:
  - CosmoPhotoz
  - Python
  - seaborn
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# multi-sample-photo-z-comparison-analysis

## Summary

Assembles a merged comparison table of photometric-redshift quality metrics (bias, rms, catastrophic outlier rates) for the gamma-family GLM method versus published baseline photo-z codes (Le PHARE, BPZ, ANNz, TPZ, BMA), tabulated per sample (PHAT0, SDSS) and per number of retained PCs.

## When to use

Use when benchmarking a photo-z method's catastrophic outlier rates (O(a): |zphot-zspec|>0.10; O(b): |zphot-zspec|/(1+zspec)>0.10) against published codes across simulated (PHAT0) and real (SDSS DR10) samples.

## When NOT to use

- The goal is to re-run the GLM photo-z fit itself rather than tabulate/compare already-reported metrics — use the CosmoPhotoz fitting workflow instead
- The comparison targets metrics not tabulated in the article (e.g., k-fold cross-validated under/over-estimation of photo-z), which the authors explicitly declare out of scope
- Only one sample and one method are involved, so a multi-sample, multi-code merged comparison table adds no value

## Inputs

- GLM photo-z evaluation metrics (catastrophic outlier rates, bias, rms) from the article's results tables for PHAT0 and SDSS samples across PC configurations
- Published baseline photo-z code metrics (ANNz, BPZ, Le PHARE, EAZY, TPZ, BMA) as tabulated in the article's results section
- Outlier definition per entry: O(a) Δz=|zphot−zspec|>0.10 or O(b) Δz=|zphot−zspec|/(1+zspec)>0.10

## Outputs

- photoz_metrics_comparison.csv — merged pandas DataFrame with columns for sample (PHAT0/SDSS), method (GLM vs baseline codes), and reported bias/rms/outlier metrics under O(a) and O(b) definitions

## How to apply

Retrieve the GLM photo-z diagnostics from the article's results tables: gamma-family GLM with log link fitted on robust-PCA principal components, using a 1:9 train/test split (33,904 training / 135,616 testing galaxies for PHAT0) and ≥6 PCs to retain ~99.5% of variance. Retrieve bias, rms, and outlier values for baseline codes (ANNz, BPZ, Le PHARE, EAZY, TPZ, BMA) exactly as tabulated, keeping whichever outlier definition the article reports per entry — O(a) with Δz=|zphot−zspec|>0.10 or O(b) with Δz=|zphot−zspec|/(1+zspec)>0.10. Assemble rows in Python (pandas) with columns for sample (PHAT0/SDSS), method (GLM vs baseline), and reported metrics, and save as photoz_metrics_comparison.csv. Validate that every tabulated method-sample pair has a row, both O(a)/O(b) columns are present, and the GLM outlier rates match the reported values (e.g., 1.367% at 10 PCs, 2.511% at 8 PCs, 4.438% at 6 PCs on PHAT0; 6.44% at 5 PCs, 8.09% at 4 PCs on SDSS).

## Related tools

- **Python** (pandas-based assembly of the merged photoz_metrics_comparison.csv table from the article's reported metrics)
- **CosmoPhotoz** (source package implementing the gamma-family GLM photo-z estimation whose reported catastrophic outlier rates are compared against baseline codes) — https://github.com/COINtoolbox/COSMOPhotoz/
- **seaborn** (statistical visualization library cited by the article for plotting the photo-z comparison diagnostics) — https://github.com/mwaskom/seaborn

## Examples

```
import pandas as pd; rows=[{'sample':'PHAT0','method':'GLM (10 PCs)','outlier_pct':1.367},{'sample':'PHAT0','method':'GLM (6 PCs)','outlier_pct':4.438},{'sample':'SDSS','method':'GLM (5 PCs)','outlier_pct':6.44}]; pd.DataFrame(rows).to_csv('photoz_metrics_comparison.csv',index=False)
```

## Evaluation signals

- Row count equals the number of tabulated method-sample pairs, with a row for every baseline code and every GLM PC configuration
- Both O(a) (|zphot−zspec|>0.10) and O(b) (|zphot−zspec|/(1+zspec)>0.10) columns are present in the CSV
- GLM outlier rates reproduce reported values: 1.367% (10 PCs), 2.511% (8 PCs), 4.438% (6 PCs) on PHAT0 and 6.44% (5 PCs), 8.09% (4 PCs) on SDSS
- Simulated-data GLM outlier rates fall near ~1% and real-data (SDSS) rates near ~2%, consistent with the abstract's claim
- Each metric entry carries the outlier definition actually used by the article for that method, with no silently mixed definitions

## Limitations

- The article does not perform a k-fold cross-validation analysis of photo-z under/over-estimation, so such bias diagnostics cannot be populated for the GLM from this source
- Slight differences exist between the R and Python implementations in the PCA SVD solving and GLM minimisation steps, which are not fully reconciled and may affect reproducibility of the tabulated metrics
- Baseline-code metrics must be copied under whichever outlier definition each code's original publication used; entries are not uniformly defined across O(a) and O(b)

## Evidence

- [results] Table 1 outlier rates across PC configurations: [evidence span withheld — anchor status: not_found]
- [methods] Catastrophic error metric definition: [evidence span withheld — anchor status: not_found]
