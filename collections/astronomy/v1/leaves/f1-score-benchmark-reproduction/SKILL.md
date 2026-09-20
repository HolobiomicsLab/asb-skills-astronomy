---
name: f1-score-benchmark-reproduction
description: Use when the goal is to verify or benchmark a machine-learning classification
  pipeline against published per-class F1-scores on SDSS+WISE photometry, or to establish
  a reproducible galaxy/QSO/star classification baseline before modifying the method.
when_to_use_negative:
- The input is non-tabular data (e.g. images or spectra) — SHEEP was built for tabular
  data, though other tabular data such as radio fluxes/magnitudes are compatible.
- You are applying the models to a different dataset without re-optimising — the README
  warns the models are optimised for SDSS DR15+WISE and hyper-parameters must be changed
  accordingly.
- You need per-source probabilistic class membership with meta-learner correction
  rather than multi-class labels — the paper's second methodology ('clf_1vsall.py',
  one versus all with meta-learner correction) is a separate pipeline.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_3379
tools:
- name: SHEEP
  role: Overall pipeline whose multi-class classification results are being reproduced
  repo: https://github.com/pedro-acunha/SHEEP
- name: XGBoost
  role: One of three base learners whose outputs are combined by the ensemble classifier
  repo: https://github.com/pedro-acunha/SHEEP
- name: LightGBM
  role: One of three base learners whose outputs are combined by the ensemble classifier
  repo: https://github.com/pedro-acunha/SHEEP
- name: CatBoost
  role: One of three base learners whose outputs are combined by the ensemble classifier
  repo: https://github.com/pedro-acunha/SHEEP
- name: FLAML
  role: Hyper-parameter optimisation re-run on the SDSS DR15+WISE input data before
    training
  repo: https://github.com/pedro-acunha/SHEEP
- name: Python
  role: Implementation language used to run the pipeline scripts
- name: Jupyter Notebook
  role: Recommended environment for visualisation and interactivity during reproduction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202243135
    title: Photometric redshift-aided classification using ensemble learning
  source_content_hash: sha256:7444b884f1e3dcee87edf1f785a1780513b72cf7b4d2a002060da7dd3c60187d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pedro-acunha/SHEEP
  tools:
  - CatBoost
  - FLAML
  - Jupyter Notebook
  - LightGBM
  - Python
  - SHEEP
  - XGBoost
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_3379
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# f1-score-benchmark-reproduction

## Summary

Reproduce the SHEEP multi-class (galaxy/quasar/star) F1 benchmark of 0.992/0.967/0.985 on the 3.5-million-source SDSS DR15+WISE photometric catalogue by re-running the published pipeline: CasJobs data retrieval, feature engineering, chain-regressor photo-z, FLAML hyper-parameter optimisation, and XGBoost/LightGBM/CatBoost ensemble classification.

## When to use

Use when the goal is to verify or benchmark a machine-learning classification pipeline against published per-class F1-scores on SDSS+WISE photometry, or to establish a reproducible galaxy/QSO/star classification baseline before modifying the method.

## When NOT to use

- The input is non-tabular data (e.g. images or spectra) — SHEEP was built for tabular data, though other tabular data such as radio fluxes/magnitudes are compatible.
- You are applying the models to a different dataset without re-optimising — the README warns the models are optimised for SDSS DR15+WISE and hyper-parameters must be changed accordingly.
- You need per-source probabilistic class membership with meta-learner correction rather than multi-class labels — the paper's second methodology ('clf_1vsall.py', one versus all with meta-learner correction) is a separate pipeline.

## Inputs

- SDSS DR15+WISE photometric catalogue of ~3.5 million astronomical sources (retrieved via SDSS SkyServer CasJobs with the 'sql_query_casjobs' SQL queries)
- SHEEP repository scripts: sql_query_casjobs, create_data.py, photo_z.py, clf_multi.py
- FLAML hyper-parameter configurations reported in the paper

## Outputs

- Trained multi-class galaxy/QSO/star ensemble classifier (XGBoost + LightGBM + CatBoost)
- Per-class F1-scores; target benchmark: 0.992 (galaxies), 0.967 (quasars), 0.985 (stars)
- Estimated photometric redshifts used as classification features

## How to apply

Retrieve the SDSS DR15+WISE photometry for the 3.5-million-source sample using the SQL queries in the repository's 'sql_query_casjobs' file via SDSS SkyServer CasJobs (https://skyserver.sdss.org/CasJobs/). Run 'create_data.py' to perform the feature engineering on the tabular photometry, adapting features only if deliberately changing the dataset. Estimate photometric redshifts with the chain-regressor approach in 'photo_z.py' and append the photo-z values as an additional classification feature — the paper reports this yields significant performance improvements. Re-run FLAML hyper-parameter optimisation on the SDSS DR15+WISE input data (the README recommends re-running FLAML every time the input data changes, since the models are optimised for SDSS DR15+WISE). Train the multi-class classifier from 'clf_multi.py', which combines XGBoost, LightGBM, and CatBoost outputs into a stronger classifier, then evaluate and compute per-class F1-scores.

## Related tools

- **SHEEP** (Overall pipeline whose multi-class classification results are being reproduced) — https://github.com/pedro-acunha/SHEEP
- **XGBoost** (One of three base learners whose outputs are combined by the ensemble classifier) — https://github.com/pedro-acunha/SHEEP
- **LightGBM** (One of three base learners whose outputs are combined by the ensemble classifier) — https://github.com/pedro-acunha/SHEEP
- **CatBoost** (One of three base learners whose outputs are combined by the ensemble classifier) — https://github.com/pedro-acunha/SHEEP
- **FLAML** (Hyper-parameter optimisation re-run on the SDSS DR15+WISE input data before training) — https://github.com/pedro-acunha/SHEEP
- **Python** (Implementation language used to run the pipeline scripts)
- **Jupyter Notebook** (Recommended environment for visualisation and interactivity during reproduction)

## Examples

```
python clf_multi.py  # after create_data.py, photo_z.py, and a FLAML re-run on the SDSS DR15+WISE table from CasJobs
```

## Evaluation signals

- Per-class F1-scores match the reported benchmark: 0.992 for galaxies, 0.967 for quasars, and 0.985 for stars.
- SHEEP's three-class F1-scores outperform the recent RandomForest-based classification approach on an essentially identical data set.
- Classification metrics are computed with the photometric redshift included as a feature — omitting the photo-z feature should produce measurably worse scores, per the reported significant improvement.
- FLAML has been re-run on the current input data rather than reusing stale hyper-parameters, per the README recommendation.
- The evaluation is on the full ~3.5-million-source SDSS+WISE sample, not a subsample.

## Limitations

- The models are optimised for SDSS DR15+WISE; using a different dataset requires changing the hyper-parameters (README: 'If you use it with a different dataset, please remember that the models are optimised for SDSS DR15+WISE.').
- FLAML hyper-parameters should be re-run every time changes are made to the input data.
- F1-scores of 0.992/0.967/0.985 were obtained with the multi-class methodology; the one-vs-all methodology with meta-learner correction is a distinct pipeline in 'clf_1vsall.py'.
- The pipeline was built for tabular data allowing sparse data; incompatible data types require adaptation.

## Evidence

- [abstract] The resulting F1 -scores are as follows: 0.992 for galaxies; 0.967 for quasars; and 0.985 for stars.: "The resulting F1 -scores are as follows: 0.992 for galaxies; 0.967 for quasars; and 0.985 for stars."
- [abstract] SHEEP first estimates photometric redshifts, which are then placed into the data set as an additional feature for classification model training: "prior to performing the classification, SHEEP first estimates photometric redshifts, which are then placed into the data set as an additional feature for classification model training"
- [methods] The hyper-parameters obtained using FLAML are described. It is recommended to re-run FLAML everytime changes are made in the input data.: "The hyper-parameters obtained using FLAML are described. It is recommended to re-run FLAML everytime changes are made in the input data."
- [readme] If you use it with a different dataset, please remember that the models are optimised for SDSS DR15+WISE. Do not forget to change them according to your dataset.: "If you use it with a different dataset, please remember that the models are optimised for SDSS DR15+WISE. Do not forget to change them according to your dataset."
- [readme] To create the data you can use the SQL queries provided in the file "sql_query_casjobs". The data can be retrieved from https://skyserver.sdss.org/CasJobs/: "To create the data you can use the SQL queries provided in the file "sql_query_casjobs". The data can be retrieved from https://skyserver.sdss.org/CasJobs/"
