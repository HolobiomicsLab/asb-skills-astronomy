---
name: gradient-boosting-ensemble-training
description: Use when classifying large tabular photometric catalogues (e.g. ~3.5M
  SDSS+WISE sources) into galaxies, quasars, and stars and a single gradient-boosting
  model is insufficient — an ensemble of XGBoost/LightGBM/CatBoost outputs is needed.
when_to_use_negative:
- Input is not tabular data — SHEEP was built for tabular data (with sparse-data support),
  so non-tabular inputs like images are out of scope
- Using a different dataset without re-tuning — models are optimised for SDSS DR15+WISE
  and FLAML should be re-run when input data change; do not reuse the paper's hyper-parameters
  unmodified
- The task is photometric redshift estimation rather than classification — use the
  chain-regressor approach (photo_z.py) instead
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3315
tools:
- name: SHEEP
  role: Pipeline providing the ensemble mechanism that combines the outputs of the
    three gradient-boosting learners (clf_multi.py / clf_1vsall.py)
  repo: github.com/pedro-acunha/SHEEP
- name: XGBoost
  role: Base learner whose outputs are combined into the stronger classifier
  repo: github.com/pedro-acunha/SHEEP
- name: LightGBM
  role: Base learner whose outputs are combined into the stronger classifier
  repo: github.com/pedro-acunha/SHEEP
- name: CatBoost
  role: Base learner whose outputs are combined into the stronger classifier
  repo: github.com/pedro-acunha/SHEEP
- name: FLAML
  role: Hyper-parameter optimisation for the base learners; recommended to re-run
    whenever input data change
  repo: github.com/pedro-acunha/SHEEP
- name: Python
  role: Implementation language for training and evaluation
  repo: github.com/pedro-acunha/SHEEP
- name: Jupyter Notebook
  role: Recommended environment for visualisation and interactivity
  repo: github.com/pedro-acunha/SHEEP
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202243135
    title: Photometric redshift-aided classification using ensemble learning
  source_content_hash: sha256:48cc6ae4747a86a48681e1d1db675fa40e1eb7f6f2744482acdefa4ea8043a94
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CatBoost
  - FLAML
  - Jupyter Notebook
  - LightGBM
  - Python
  - SHEEP
  - XGBoost
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# gradient-boosting-ensemble-training

## Summary

Train a stronger astronomical source classifier (galaxies/quasars/stars) by combining the outputs of XGBoost, LightGBM, and CatBoost gradient-boosting models, with hyper-parameters tuned by FLAML, as implemented in the SHEEP pipeline.

## When to use

Use when classifying large tabular photometric catalogues (e.g. ~3.5M SDSS+WISE sources) into galaxies, quasars, and stars and a single gradient-boosting model is insufficient — an ensemble of XGBoost/LightGBM/CatBoost outputs is needed.

## When NOT to use

- Input is not tabular data — SHEEP was built for tabular data (with sparse-data support), so non-tabular inputs like images are out of scope
- Using a different dataset without re-tuning — models are optimised for SDSS DR15+WISE and FLAML should be re-run when input data change; do not reuse the paper's hyper-parameters unmodified
- The task is photometric redshift estimation rather than classification — use the chain-regressor approach (photo_z.py) instead

## Inputs

- Feature-engineered SDSS DR15+WISE tabular photometry of ~3.5 million astronomical sources (from create_data.py, optionally augmented with chain-regressor photo-z values)
- FLAML-optimised hyper-parameters for XGBoost, LightGBM, and CatBoost
- SQL queries (sql_query_casjobs) for retrieving the catalogue from SkyServer CasJobs

## Outputs

- Combined ensemble classifier (e.g. serialised model artifact) predicting galaxy/QSO/star classes
- Per-class F1-scores on held-out data (reported: 0.992 galaxies, 0.967 quasars, 0.985 stars)
- Per-class predictions for galaxies, quasars, and stars

## How to apply

Load the engineered SDSS DR15+WISE tabular feature set (ideally including the photometric-redshift column produced by the chain-regressor in photo_z.py, since adding photo-z yields significant F1 improvements). Retrieve hyper-parameters from the FLAML optimisation described in the paper, re-running FLAML whenever the input data change, as the models are optimised for SDSS DR15+WISE. Fit XGBoost, LightGBM, and CatBoost and apply SHEEP's ensemble mechanism to combine the three learners' outputs, via either the multi-class methodology (clf_multi.py) or the one-vs-all methodology with meta-learner correction (clf_1vsall.py). Evaluate per-class F1 on a held-out test set: the reported benchmark is 0.992 (galaxies), 0.967 (quasars), 0.985 (stars) on 3.5 million sources. The pipeline handles sparse tabular data and supports other tabular photometry (e.g. radio fluxes/magnitudes) with re-tuned hyper-parameters.

## Related tools

- **SHEEP** (Pipeline providing the ensemble mechanism that combines the outputs of the three gradient-boosting learners (clf_multi.py / clf_1vsall.py)) — github.com/pedro-acunha/SHEEP
- **XGBoost** (Base learner whose outputs are combined into the stronger classifier) — github.com/pedro-acunha/SHEEP
- **LightGBM** (Base learner whose outputs are combined into the stronger classifier) — github.com/pedro-acunha/SHEEP
- **CatBoost** (Base learner whose outputs are combined into the stronger classifier) — github.com/pedro-acunha/SHEEP
- **FLAML** (Hyper-parameter optimisation for the base learners; recommended to re-run whenever input data change) — github.com/pedro-acunha/SHEEP
- **Python** (Implementation language for training and evaluation) — github.com/pedro-acunha/SHEEP
- **Jupyter Notebook** (Recommended environment for visualisation and interactivity) — github.com/pedro-acunha/SHEEP

## Examples

```
python clf_multi.py  # train XGBoost+LightGBM+CatBoost ensemble on SDSS DR15+WISE features with FLAML hyper-parameters
```

## Evaluation signals

- Per-class F1-scores on held-out data reach the reported benchmark: 0.992 (galaxies), 0.967 (quasars), 0.985 (stars) on 3.5 million SDSS+WISE sources
- Ensemble outperforms individual base learners and a RandomForest-based classification approach on an essentially identical data set in three-class F1
- With-photo-z run shows significant F1 improvement over a without-photo-z ablation run
- FLAML has been re-run after any change to the input features before fitting the base learners
- Both classification methodologies (multi-class and one-vs-all with meta-learner correction) can be trained and their per-class scores compared

## Limitations

- Models are optimised for SDSS DR15+WISE; hyper-parameters must be changed when using a different dataset
- FLAML hyper-parameter search should be re-run every time changes are made in the input data
- Only tabular data (with sparse-data support) are compatible; other photometry types must be tabular (e.g. radio fluxes/magnitudes)

## Evidence

- [abstract] combines the outputs from the XGBoost, LightGBM, and CatBoost learning algorithms to create stronger classifiers: "combines the outputs from the XGBoost, LightGBM, and CatBoost learning algorithms to create stronger classifiers"
- [abstract] SHEEP contains two distinct classification methodologies: (i) Multi-class and (ii) one versus all with correction by a meta-learner.: "SHEEP contains two distinct classification methodologies: (i) Multi-class and (ii) one versus all with correction by a meta-learner."
- [readme] The hyper-parameters obtained using FLAML are described. It is recommended to re-run FLAML everytime changes are made in the input data.: "The hyper-parameters obtained using FLAML are described. It is recommended to re-run FLAML everytime changes are made in the input data."
- [readme] If you use it with a different dataset, please remember that the models are optimised for SDSS DR15+WISE.: "If you use it with a different dataset, please remember that the models are optimised for SDSS DR15+WISE. Do not forget to change them according to your dataset."
- [readme] It was built to deal with tabular data, while allowing the use of sparse data.: "It was built to deal with tabular data, while allowing the use of sparse data."
- [abstract] The resulting F1 -scores are as follows: 0.992 for galaxies; 0.967 for quasars; and 0.985 for stars.: "The resulting F1 -scores are as follows: 0.992 for galaxies; 0.967 for quasars; and 0.985 for stars."
