---
name: published-benchmark-comparison-for-source-classification
description: Use when you need to verify that a new astronomical source-classification
  pipeline outperforms a published baseline (e.g. RandomForest) on an essentially
  identical photometric data set, using per-class F1-scores for galaxy/QSO/star classes.
when_to_use_negative:
- The input data set differs substantially from SDSS DR15+WISE photometry — the SHEEP
  models and hyper-parameters are optimised for SDSS DR15+WISE, so the benchmark would
  not be comparable
- The baseline you are comparing against was trained on different features or a different
  data set, making the 'essentially identical data set' condition unmet
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: SHEEP
  role: Reference pipeline whose published per-class F1-scores (0.992 galaxies, 0.967
    quasars, 0.985 stars) serve as the benchmark
  repo: github.com/pedro-acunha/SHEEP
- name: RandomForest
  role: Baseline classifier trained on identical features/data whose per-class F1-scores
    are computed for comparison
- name: Python
  role: Implementation language for training the baseline and computing F1-scores
- name: Jupyter Notebook
  role: Recommended environment for running and visualising the comparison
- name: SDSS SkyServer CasJobs
  role: Source of the SDSS DR15+WISE photometric data via the 'sql_query_casjobs'
    SQL queries
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202243135
    title: Photometric redshift-aided classification using ensemble learning
  source_content_hash: sha256:0d71b0fcb08c9169a1ed26c735cd968c874d33db890ceb7adb9692a53cdb3bfa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter Notebook
  - Python
  - RandomForest
  - SDSS SkyServer CasJobs
  - SHEEP
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# published-benchmark-comparison-for-source-classification

## Summary

Reproduces a benchmark comparison in which SHEEP (an ensemble of XGBoost, LightGBM, and CatBoost with photometric-redshift features) is evaluated against a recent RandomForest-based classification approach on the SDSS+WISE 3.5-million-source data set, using per-class F1-scores for galaxies, quasars, and stars.

## When to use

Use when you need to verify that a new astronomical source-classification pipeline outperforms a published baseline (e.g. RandomForest) on an essentially identical photometric data set, using per-class F1-scores for galaxy/QSO/star classes.

## When NOT to use

- The input data set differs substantially from SDSS DR15+WISE photometry — the SHEEP models and hyper-parameters are optimised for SDSS DR15+WISE, so the benchmark would not be comparable
- The baseline you are comparing against was trained on different features or a different data set, making the 'essentially identical data set' condition unmet

## Inputs

- SDSS DR15+WISE photometric dataset of 3.5 million sources (retrieved via SQL queries in 'sql_query_casjobs' from SDSS SkyServer CasJobs)
- Feature-engineered table produced by 'create_data.py', with galaxy/QSO/star labels
- SHEEP's published per-class F1-scores (0.992 galaxies, 0.967 quasars, 0.985 stars)

## Outputs

- Per-class F1-scores of the RandomForest baseline for galaxies, quasars, and stars
- Comparison table (e.g. sheep_vs_randomforest_f1_comparison.csv) showing SHEEP vs RandomForest F1 per class

## How to apply

Retrieve the SDSS DR15+WISE photometric data set (3.5 million astronomical sources) via the SQL queries in the 'sql_query_casjobs' file from SDSS SkyServer CasJobs (https://skyserver.sdss.org/CasJobs/). Apply the same feature-engineering procedure implemented in 'create_data.py' so the RandomForest baseline is trained on features identical to those used by SHEEP, keeping the models optimised for SDSS DR15+WISE. Train the RandomForest baseline to predict galaxy, QSO, and star labels in Python/Jupyter Notebook, then compute its per-class F1-scores. Compare against SHEEP's reported F1-scores — 0.992 for galaxies, 0.967 for quasars, 0.985 for stars — verifying SHEEP exceeds the baseline on all three classes, and save the comparison as a table (e.g. sheep_vs_randomforest_f1_comparison.csv).

## Related tools

- **SHEEP** (Reference pipeline whose published per-class F1-scores (0.992 galaxies, 0.967 quasars, 0.985 stars) serve as the benchmark) — github.com/pedro-acunha/SHEEP
- **RandomForest** (Baseline classifier trained on identical features/data whose per-class F1-scores are computed for comparison)
- **Python** (Implementation language for training the baseline and computing F1-scores)
- **Jupyter Notebook** (Recommended environment for running and visualising the comparison)
- **SDSS SkyServer CasJobs** (Source of the SDSS DR15+WISE photometric data via the 'sql_query_casjobs' SQL queries)

## Examples

```
jupyter notebook sheep_vs_randomforest_f1_comparison.ipynb  # train RandomForest baseline on create_data.py features, compute per-class F1, save CSV
```

## Evaluation signals

- RandomForest baseline F1-score is below SHEEP's reported value in every class: < 0.992 for galaxies, < 0.967 for quasars, < 0.985 for stars
- The baseline is trained on the same feature set produced by 'create_data.py' as SHEEP (feature parity verified)
- The evaluation data set is the SDSS+WISE set of 3.5 million astronomical sources, matching the published benchmark scale
- The output comparison table contains per-class F1 values for both SHEEP and RandomForest for all three classes (galaxies, quasars, stars)

## Limitations

- SHEEP's models are optimised for SDSS DR15+WISE; using a different data set requires re-tuning hyper-parameters (re-run FLAML), which breaks strict comparability with the published scores
- The comparison is only valid when the RandomForest baseline uses 'an essentially identical data set'; differences in feature engineering or data retrieval invalidate the benchmark

## Evidence

- [abstract] SHEEP outperforms a recent RandomForest-based classification approach on an essentially identical data set in terms of the three-class F1-scores: "SHEEP is found to outperform a recent RandomForest-based classification approach using an essentially identical data set."
- [abstract] Published per-class F1-scores benchmark: "The resulting F1 -scores are as follows: 0.992 for galaxies; 0.967 for quasars; and 0.985 for stars."
- [methods] Data retrieval via CasJobs SQL queries: "To create the data you can use the SQL queries provided in the file "sql_query_casjobs". The data can be retrieved from https://skyserver.sdss.org/CasJobs/ ."
- [readme] Feature engineering for identical baseline features: "All the feature engineering process can be seen in the file "create_data.py"."
- [readme] Models optimised for SDSS DR15+WISE: "If you use it with a different dataset, please remember that the models are optimised for SDSS DR15+WISE."
