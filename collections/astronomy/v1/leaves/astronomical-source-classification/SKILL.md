---
name: astronomical-source-classification
description: Use when you have tabular photometry (e.g. SDSS DR15+WISE magnitudes/fluxes,
  possibly sparse) for millions of sources and need 3-class classification into galaxies,
  quasars, and stars with per-class F1 targets.
when_to_use_negative:
- Input is not tabular data — the architecture is built specifically for tabular data
  (e.g. imaging or spectra would need a different approach).
- You are using a non-SDSS/WISE dataset without re-optimising — the models are optimised
  for SDSS DR15+WISE and the README warns to change hyper-parameters according to
  your dataset.
- You only have photo-z estimation as the goal — this skill covers classification
  downstream of the photo-z step (photo_z.py), not regression itself.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: SHEEP
  role: Overall pipeline that estimates photo-z and combines base learners into the
    stronger classifier (clf_multi.py / clf_1vsall.py)
  repo: github.com/pedro-acunha/SHEEP
- name: XGBoost
  role: Base learning algorithm whose outputs are combined in the ensemble
- name: LightGBM
  role: Base learning algorithm whose outputs are combined in the ensemble
- name: CatBoost
  role: Base learning algorithm whose outputs are combined in the ensemble
- name: FLAML
  role: Hyper-parameter optimisation for the base learners; recommended to re-run
    whenever input data change
- name: Python
  role: Implementation language for the pipeline
- name: Jupyter Notebook
  role: Recommended environment for visualisation and interactivity
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202243135
    title: Photometric redshift-aided classification using ensemble learning
  source_content_hash: sha256:bb417bff66891adef00eb7946e0c619465f4d0dae88d578751d6ec5b6ce270a1
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# astronomical-source-classification

## Summary

SHEEP-style classification of astronomical sources into galaxies, quasars, and stars from SDSS+WISE photometry, using XGBoost/LightGBM/CatBoost ensembles with photo-z as an added feature. Supports both multi-class and one-vs-all (meta-learner corrected) setups.

## When to use

Use when you have tabular photometry (e.g. SDSS DR15+WISE magnitudes/fluxes, possibly sparse) for millions of sources and need 3-class classification into galaxies, quasars, and stars with per-class F1 targets.

## When NOT to use

- Input is not tabular data — the architecture is built specifically for tabular data (e.g. imaging or spectra would need a different approach).
- You are using a non-SDSS/WISE dataset without re-optimising — the models are optimised for SDSS DR15+WISE and the README warns to change hyper-parameters according to your dataset.
- You only have photo-z estimation as the goal — this skill covers classification downstream of the photo-z step (photo_z.py), not regression itself.

## Inputs

- SDSS DR15+WISE photometric dataset of ~3.5 million sources (retrieved via sql_query_casjobs SQL queries from SkyServer CasJobs)
- Engineered tabular feature set from create_data.py, including the photometric-redshift column produced by photo_z.py
- FLAML-tuned hyper-parameters for XGBoost, LightGBM, and CatBoost

## Outputs

- Per-class predictions for galaxies, quasars, and stars (e.g. ensemble_predictions.csv)
- Serialised combined ensemble model artifact (e.g. ensemble_model.pkl)
- Per-class F1 scores on the held-out test set (expected ~0.992 galaxies, ~0.967 quasars, ~0.985 stars)

## How to apply

Retrieve SDSS+WISE photometry via SQL queries in SkyServer CasJobs (sql_query_casjobs), then run feature engineering (create_data.py). Estimate photometric redshifts with the chain-regressor (photo_z.py) and append the photo-z column to the feature set before classification — this drives significant F1 gains (0.992 galaxies, 0.967 quasars, 0.985 stars on 3.5M sources). Fit XGBoost, LightGBM, and CatBoost with FLAML-tuned hyper-parameters (re-run FLAML whenever the input data change), then combine the three learners' outputs either multi-class (clf_multi.py) or one-vs-all with meta-learner correction (clf_1vsall.py). Compare against a RandomForest baseline to confirm the ensemble outperforms it on the same data set.

## Related tools

- **SHEEP** (Overall pipeline that estimates photo-z and combines base learners into the stronger classifier (clf_multi.py / clf_1vsall.py)) — github.com/pedro-acunha/SHEEP
- **XGBoost** (Base learning algorithm whose outputs are combined in the ensemble)
- **LightGBM** (Base learning algorithm whose outputs are combined in the ensemble)
- **CatBoost** (Base learning algorithm whose outputs are combined in the ensemble)
- **FLAML** (Hyper-parameter optimisation for the base learners; recommended to re-run whenever input data change)
- **Python** (Implementation language for the pipeline)
- **Jupyter Notebook** (Recommended environment for visualisation and interactivity)

## Examples

```
python clf_multi.py  # train the multi-class XGBoost+LightGBM+CatBoost ensemble with FLAML-tuned hyper-parameters on the photo-z-aided SDSS+WISE feature set
```

## Evaluation signals

- Per-class F1-scores on the held-out test set reach the reported levels: 0.992 (galaxies), 0.967 (quasars), 0.985 (stars).
- The with-photo-z run shows a significant F1 improvement over an identical ablation run with the photo-z column removed (with FLAML re-run for the changed input).
- SHEEP's F1-scores outperform a recent RandomForest-based classification approach on an essentially identical data set.
- Ensemble model is serialised and predictions cover all three classes (galaxies, quasars, stars) on the held-out set.
- Data pipeline reproduces ~3.5 million SDSS+WISE sources from the CasJobs SQL queries.

## Limitations

- Models are optimised for SDSS DR15+WISE; when using a different dataset the FLAML-tuned hyper-parameters must be changed accordingly.
- Adding photo-z as a feature is a prerequisite for the reported performance — omitting it degrades classification.
- The pipeline assumes tabular input; sparse data are supported, but non-tabular modalities are outside the architecture's scope.

## Evidence

- [abstract] We present SHEEP, a new machine learning approach to the classic problem of astronomical source classification: "We present SHEEP, a new machine learning approach to the classic problem of astronomical source classification"
- [abstract] combines the outputs from the XGBoost, LightGBM, and CatBoost learning algorithms to create stronger classifiers: "combines the outputs from the XGBoost, LightGBM, and CatBoost learning algorithms to create stronger classifiers"
- [abstract] prior to performing the classification, SHEEP first estimates photometric redshifts, which are then placed into the data set as an additional feature: "prior to performing the classification, SHEEP first estimates photometric redshifts, which are then placed into the data set as an additional feature"
- [methods] Multi-class: see file "clf_multi.py".: [evidence span withheld — anchor status: not_found]
- [readme] The hyper-parameters obtained using FLAML are described. It is recommended to re-run FLAML everytime changes are made in the input data.: "The hyper-parameters obtained using FLAML are described. It is recommended to re-run FLAML everytime changes are made in the input data."
- [readme] To create the data you can use the SQL queries provided in the file "sql_query_casjobs".: "To create the data you can use the SQL queries provided in the file "sql_query_casjobs". The data can be retrieved from https://skyserver.sdss.org/CasJobs/ ."
- [readme] If you use it with a different dataset, please remember that the models are optimised for SDSS DR15+WISE.: "If you use it with a different dataset, please remember that the models are optimised for SDSS DR15+WISE. Do not forget to change them according to your dataset."
