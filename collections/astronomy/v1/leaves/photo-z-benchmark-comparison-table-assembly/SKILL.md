---
name: photo-z-benchmark-comparison-table-assembly
description: Use when reported photo-z quality metrics (outlier rates, bias, rms)
  must be compared against published benchmark codes, e.g. to reproduce or extend
  the article's Table 1 comparison.
when_to_use_negative:
- The raw photometric data must be re-analysed to produce new photo-z estimates —
  this skill only tabulates already-reported metrics, it does not refit GLMs or run
  baseline codes
- The source article reports metrics under only one outlier definition and no cross-definition
  harmonisation is required — a simple single-column table suffices
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Python
  role: pandas is used to assemble and validate the merged comparison table and save
    it as photoz_metrics_comparison.csv
- name: CosmoPhotoz
  role: the GLM photo-z package whose reported evaluation metrics (catastrophic outlier
    rates on PHAT0 and SDSS) populate the GLM rows of the comparison table
  repo: https://github.com/COINtoolbox/COSMOPhotoz/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2015.01.002
    title: 'The overlooked potential of Generalized Linear Models in astronomy-II:
      Gamma regression and photometric redshifts'
  source_content_hash: sha256:591d8b63b9e3d998a02741da88cb94219e3106e235c3054a8a3c40b19610afa7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/COINtoolbox/COSMOPhotoz
  tools:
  - CosmoPhotoz
  - Python
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# photo-z-benchmark-comparison-table-assembly

## Summary

Assembles a merged comparison table in pandas of GLM gamma-regression photo-z diagnostics (catastrophic outlier rates on PHAT0 and SDSS across PC configurations) against baseline template, empirical, and hybrid photo-z codes (Le PHARE, BPZ, ANNz, TPZ, BMA) as tabulated in the article.

## When to use

Use when reported photo-z quality metrics (outlier rates, bias, rms) must be compared against published benchmark codes, e.g. to reproduce or extend the article's Table 1 comparison.

## When NOT to use

- The raw photometric data must be re-analysed to produce new photo-z estimates — this skill only tabulates already-reported metrics, it does not refit GLMs or run baseline codes
- The source article reports metrics under only one outlier definition and no cross-definition harmonisation is required — a simple single-column table suffices

## Inputs

- GLM photo-z evaluation metrics (catastrophic outlier rates under O(a) and O(b) definitions) for PHAT0 and SDSS from the article's results tables
- Reported bias, rms, and outlier values of baseline photo-z codes (ANNz, BPZ, Le PHARE, EAZY, TPZ, BMA) as tabulated in the article's results section

## Outputs

- photoz_metrics_comparison.csv — a pandas-assembled table with columns for sample (PHAT0/SDSS), method (GLM vs baseline codes), and reported metrics including O(a)/O(b) outlier columns

## How to apply

Retrieve the GLM evaluation metrics for the PHAT0 sample (gamma family with log link, >=6 PCs retaining 99.5% of variance, 1:9 train/test split) and the SDSS sample exactly as reported, e.g. catastrophic errors of 1.367% (10 PCs), 2.511% (8 PCs), 4.438% (6 PCs) on PHAT0 and 6.44% (5 PCs), 8.09% (4 PCs) on SDSS. Then retrieve the reported bias, rms, and outlier values for each baseline code (ANNz, BPZ, Le PHARE, TPZ, BMA, etc.) verbatim from the article's results tables. Record each entry under both outlier definitions — O(a) with Δz = |zphot − zspec| > 0.10 and O(b) with Δz = |zphot − zspec|/(1+zspec) > 0.10 — keeping whichever definition the source reports per entry. Assemble the merged table in Python with pandas, with columns for sample (PHAT0/SDSS), method (GLM vs baseline), and reported metrics, and save as photoz_metrics_comparison.csv.

## Related tools

- **Python** (pandas is used to assemble and validate the merged comparison table and save it as photoz_metrics_comparison.csv)
- **CosmoPhotoz** (the GLM photo-z package whose reported evaluation metrics (catastrophic outlier rates on PHAT0 and SDSS) populate the GLM rows of the comparison table) — https://github.com/COINtoolbox/COSMOPhotoz/

## Examples

```
import pandas as pd; df = pd.DataFrame(rows, columns=['sample','method','bias','rms','outlier_Oa','outlier_Ob']); df.to_csv('photoz_metrics_comparison.csv', index=False)
```

## Evaluation signals

- The table contains a row for every method and sample tabulated in the article's results section (GLM configurations plus each baseline code)
- Both O(a) (Δz > 0.10) and O(b) (Δz/(1+zspec) > 0.10) outlier columns are present
- GLM outlier rates reproduce the reported values: 1.367% (10 PCs), 2.511% (8 PCs), 4.438% (6 PCs) on PHAT0 and 6.44% (5 PCs), 8.09% (4 PCs) on SDSS
- Sanity check: GLM rates are of order ~1% for simulated (PHAT0) data and ~2% for real data, comparable with current techniques
- Saved file photoz_metrics_comparison.csv loads cleanly in pandas with no missing rows for tabulated methods

## Limitations

- The article reports catastrophic outlier rates but no k-fold cross-validation analysis of under/over-estimation of photo-z, so bias-side diagnostics may be incomplete
- Slight differences exist between how the R and Python CosmoPhotoz codes solve the SVD matrices in the PCA step and minimise the GLM, so metric provenance (R vs Python implementation) should be recorded
- Baseline-code entries retain whichever outlier definition (O(a) or O(b)) the original source used, so the merged table mixes definitions across rows unless explicitly harmonised

## Evidence

- [results] PHAT0 metrics across PC configurations: [evidence span withheld — anchor status: not_found]
- [methods] catastrophic error metric: [evidence span withheld — anchor status: not_found]
