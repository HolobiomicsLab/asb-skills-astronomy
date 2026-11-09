---
name: randomforest-source-classification-baseline-training
description: Use when benchmarking a new source-classification pipeline (e.g. SHEEP)
  against an established RandomForest classifier on essentially identical tabular
  photometric data and per-class F1-scores must be reproduced.
when_to_use_negative:
- The input data is non-tabular (e.g. images or spectra) — the paper's pipeline and
  its models are optimised for tabular SDSS DR15+WISE photometry
- Your dataset differs from SDSS DR15+WISE and you cannot re-run hyper-parameter optimisation,
  since 'the models are optimised for SDSS DR15+WISE' and hyper-parameters should
  be re-tuned when input data changes
- You need the best possible classifier rather than a reproducible baseline — SHEEP's
  ensemble (XGBoost, LightGBM, CatBoost with photo-z as an additional feature) is
  reported to outperform the RandomForest approach
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3318
tools:
- name: RandomForest
  role: Baseline classification model trained to predict galaxy, QSO and star labels
    for per-class F1 comparison against SHEEP
- name: SHEEP
  role: Pipeline whose reported F1-scores (0.992 galaxies, 0.967 quasars, 0.985 stars)
    form the reference side of the comparison
  repo: https://github.com/pedro-acunha/SHEEP
- name: Python
  role: Language used to train the baseline and compute F1-scores in a Jupyter Notebook
- name: Jupyter Notebook
  role: Environment recommended for running the baseline training and comparison with
    better visualisation and interactivity
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202243135
    title: Photometric redshift-aided classification using ensemble learning
  source_content_hash: sha256:618c333f3d73cf2029a5dc930a5d86e35d19278718018585d69524fafe42452e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pedro-acunha/SHEEP
  tools:
  - Jupyter Notebook
  - Python
  - RandomForest
  - SHEEP
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# randomforest-source-classification-baseline-training

## Summary

Trains a recent RandomForest-based classification approach on SDSS DR15+WISE photometry of 3.5 million sources to serve as the baseline against which SHEEP's galaxy/quasar/star F1-scores are compared.

## When to use

Use when benchmarking a new source-classification pipeline (e.g. SHEEP) against an established RandomForest classifier on essentially identical tabular photometric data and per-class F1-scores must be reproduced.

## When NOT to use

- The input data is non-tabular (e.g. images or spectra) — the paper's pipeline and its models are optimised for tabular SDSS DR15+WISE photometry
- Your dataset differs from SDSS DR15+WISE and you cannot re-run hyper-parameter optimisation, since 'the models are optimised for SDSS DR15+WISE' and hyper-parameters should be re-tuned when input data changes
- You need the best possible classifier rather than a reproducible baseline — SHEEP's ensemble (XGBoost, LightGBM, CatBoost with photo-z as an additional feature) is reported to outperform the RandomForest approach

## Inputs

- SDSS DR15+WISE photometric tabular dataset of 3.5 million astronomical sources (retrieved via SQL queries from SDSS SkyServer CasJobs)
- Feature-engineered table produced by the 'create_data.py' procedure, identical to the features used by SHEEP
- Galaxy / QSO / star class labels

## Outputs

- Trained RandomForest classifier predicting galaxy, QSO and star classes
- Per-class F1-scores for the RandomForest baseline
- Per-class F1 comparison table (SHEEP vs RandomForest), saved as sheep_vs_randomforest_f1_comparison.csv

## How to apply

Retrieve the SDSS DR15+WISE photometric dataset (3.5 million sources) via the SQL queries in the 'sql_query_casjobs' file using SDSS SkyServer CasJobs (https://skyserver.sdss.org/CasJobs/). Run the feature-engineering procedure implemented in 'create_data.py' so the RandomForest baseline is trained on features identical to those used by SHEEP — this 'essentially identical data set' condition is what makes the F1 comparison fair. Train the RandomForest classifier in Python/Jupyter to predict the galaxy, QSO and star labels, then compute per-class F1-scores and compare them with SHEEP's reported values (0.992 galaxies, 0.967 quasars, 0.985 stars). The comparison is judged correct when the table shows which of the three classes the baseline exceeds SHEEP on; per the paper, SHEEP outperforms the RandomForest baseline for all three classes.

## Related tools

- **RandomForest** (Baseline classification model trained to predict galaxy, QSO and star labels for per-class F1 comparison against SHEEP)
- **SHEEP** (Pipeline whose reported F1-scores (0.992 galaxies, 0.967 quasars, 0.985 stars) form the reference side of the comparison) — https://github.com/pedro-acunha/SHEEP
- **Python** (Language used to train the baseline and compute F1-scores in a Jupyter Notebook)
- **Jupyter Notebook** (Environment recommended for running the baseline training and comparison with better visualisation and interactivity)

## Examples

```
python create_data.py  # build the identical feature table from the CasJobs SQL query output, then train the RandomForest baseline in a Jupyter Notebook and save sheep_vs_randomforest_f1_comparison.csv
```

## Evaluation signals

- Per-class F1-scores computed for the RandomForest baseline on the same feature table as SHEEP (built via create_data.py), enabling apples-to-apples comparison
- Reproduced comparison shows SHEEP's F1-scores of 0.992 (galaxies), 0.967 (quasars), 0.985 (stars) exceed the RandomForest baseline for all three classes
- Comparison table saved as sheep_vs_randomforest_f1_comparison.csv with exactly three class rows (galaxies, quasars, stars)
- Dataset size matches ~3.5 million SDSS+WISE sources retrieved via the CasJobs SQL queries

## Limitations

- The comparison is only valid on an 'essentially identical data set'; using different features or data breaks the fairness of the F1 comparison
- Models are optimised for SDSS DR15+WISE — if used with a different dataset, hyper-parameters must be re-tuned (FLAML is recommended to be re-run whenever input data changes)
- The RandomForest baseline is a benchmark, not the proposed method — the paper reports SHEEP outperforming it for all three classes

## Evidence

- [abstract] SHEEP is found to outperform a recent RandomForest-based classification approach using an essentially identical data set.: "SHEEP is found to outperform a recent RandomForest-based classification approach using an essentially identical data set."
- [abstract] The resulting F1 -scores are as follows: 0.992 for galaxies; 0.967 for quasars; and 0.985 for stars.: "The resulting F1 -scores are as follows: 0.992 for galaxies; 0.967 for quasars; and 0.985 for stars."
- [readme] All the feature engineering process can be seen in the file "create_data.py".: "All the feature engineering process can be seen in the file "create_data.py"."
- [methods] This work was made using Python and Jupyter Notebooks. For better visualisation and interactivity, use Jupyter Notebook.: "This work was made using Python and Jupyter Notebooks. For better visualisation and interactivity, use Jupyter Notebook."
