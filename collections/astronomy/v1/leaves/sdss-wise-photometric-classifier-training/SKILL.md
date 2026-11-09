---
name: sdss-wise-photometric-classifier-training
description: Use when classifying astronomical sources into galaxies, quasars and
  stars from tabular SDSS+WISE photometry (optionally augmented with photo-z) and
  you need the ensemble-combined classifier rather than a single base learner.
when_to_use_negative:
- Your data are not tabular — SHEEP was built to deal with tabular data (although
  other tabular data types, e.g. radio fluxes/magnitudes, are compatible)
- You changed the input data but intend to reuse the original FLAML hyper-parameters
  without re-running FLAML; the models are optimised for SDSS DR15+WISE and parameters
  must be changed for a different dataset
edam_operation: http://edamontology.org/operation_3923
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: SHEEP
  role: Pipeline providing the ensemble mechanism that combines base-learner outputs
    and the multi-class / one-vs-all classification files
  repo: https://github.com/pedro-acunha/SHEEP
- name: XGBoost
  role: Base learner whose outputs feed the ensemble classifier
- name: LightGBM
  role: Base learner whose outputs feed the ensemble classifier
- name: CatBoost
  role: Base learner whose outputs feed the ensemble classifier
- name: FLAML
  role: Hyper-parameter optimisation for the three base learners; recommended to re-run
    whenever input data change
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202243135
    title: Photometric redshift-aided classification using ensemble learning
  source_content_hash: sha256:a444bb9ff50e85b845ff8ca75644265411ee6159c3c36abd7ae480c8207f62a0
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
  edam_operation: http://edamontology.org/operation_3923
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# sdss-wise-photometric-classifier-training

## Summary

Train SHEEP's stronger ensemble classifier for galaxies, quasars, and stars by combining the outputs of XGBoost, LightGBM, and CatBoost base learners, whose hyper-parameters were optimised with FLAML on the SDSS DR15+WISE photometric feature set (with photometric redshift included as a feature).

## When to use

Use when classifying astronomical sources into galaxies, quasars and stars from tabular SDSS+WISE photometry (optionally augmented with photo-z) and you need the ensemble-combined classifier rather than a single base learner.

## When NOT to use

- Your data are not tabular — SHEEP was built to deal with tabular data (although other tabular data types, e.g. radio fluxes/magnitudes, are compatible)
- You changed the input data but intend to reuse the original FLAML hyper-parameters without re-running FLAML; the models are optimised for SDSS DR15+WISE and parameters must be changed for a different dataset

## Inputs

- Engineered SDSS DR15+WISE tabular photometry feature set from create_data.py (3.5 million sources)
- Photometric-redshift feature produced by the photo_z.py chain-regressor
- FLAML-optimised hyper-parameters for XGBoost, LightGBM, and CatBoost

## Outputs

- Serialised ensemble model artifact (e.g. ensemble_model.pkl)
- Per-class predictions for galaxies, quasars, and stars on the held-out test set (ensemble_predictions.csv)
- Per-class F1-scores on the held-out set

## How to apply

Load the engineered SDSS DR15+WISE tabular feature set produced by create_data.py, including the photometric-redshift column added by the photo_z.py chain-regressor step, since adding photo-z as a feature yields significant performance improvements. Fit XGBoost, LightGBM, and CatBoost with the hyper-parameters obtained using FLAML, re-running FLAML whenever the input data have changed because the optimised parameters are specific to SDSS DR15+WISE. Combine the three base learners' outputs via SHEEP's ensemble mechanism, choosing either the multi-class methodology (clf_multi.py) or the one-versus-all methodology with meta-learner correction (clf_1vsall.py), then serialise the combined model as an artifact (e.g. ensemble_model.pkl) and write per-class predictions on the held-out test set. Validate by computing per-class F1-scores, which should reach 0.992 for galaxies, 0.967 for quasars, and 0.985 for stars on the 3.5 million-source data set.

## Related tools

- **SHEEP** (Pipeline providing the ensemble mechanism that combines base-learner outputs and the multi-class / one-vs-all classification files) — https://github.com/pedro-acunha/SHEEP
- **XGBoost** (Base learner whose outputs feed the ensemble classifier)
- **LightGBM** (Base learner whose outputs feed the ensemble classifier)
- **CatBoost** (Base learner whose outputs feed the ensemble classifier)
- **FLAML** (Hyper-parameter optimisation for the three base learners; recommended to re-run whenever input data change)

## Examples

```
python clf_multi.py  # or python clf_1vsall.py, after fitting XGBoost/LightGBM/CatBoost with FLAML-optimised hyper-parameters on the create_data.py feature set
```

## Evaluation signals

- Per-class F1-scores on the held-out test set reach the reported levels: 0.992 (galaxies), 0.967 (quasars), 0.985 (stars)
- Ensemble F1-scores outperform the recent RandomForest-based classification approach on an essentially identical data set
- Predictions cover all three classes (galaxies, quasars, stars) on the held-out set and are written to ensemble_predictions.csv
- FLAML was re-run after any change to the input data before fitting the base learners

## Limitations

- The models are optimised for SDSS DR15+WISE; if used with a different dataset, hyper-parameters must be changed accordingly (re-run FLAML)
- Hyper-parameters obtained using FLAML must be re-run every time changes are made in the input data
- Pipeline is designed for tabular data with sparse-data support; other tabular data types are compatible but performance is not guaranteed without re-optimisation

## Evidence

- [abstract] Ensemble combination of base learners: "combines the outputs from the XGBoost, LightGBM, and CatBoost learning algorithms to create stronger classifiers"
- [abstract] Photo-z as additional feature: "SHEEP first estimates photometric redshifts, which are then placed into the data set as an additional feature for classification model"
- [abstract] Two classification methodologies: "SHEEP contains two distinct classification methodologies: (i) Multi-class and (ii) one versus all with correction by a meta-learner."
- [abstract] F1-scores reported: "The resulting F1 -scores are as follows: 0.992 for galaxies; 0.967 for quasars; and 0.985 for stars."
- [readme] FLAML re-run recommendation: "The hyper-parameters obtained using FLAML are described. It is recommended to re-run FLAML everytime changes are made in the input data."
- [readme] Models optimised for SDSS DR15+WISE: "If you use it with a different dataset, please remember that the models are optimised for SDSS DR15+WISE. Do not forget to change them according to your dataset."
