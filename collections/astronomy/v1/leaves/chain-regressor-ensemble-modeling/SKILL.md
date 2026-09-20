---
name: chain-regressor-ensemble-modeling
description: Use when a tabular astronomical catalog (e.g. SDSS+WISE photometry) lacks
  redshifts but the classification pipeline benefits from redshift as a feature; the
  photo-z must be estimated prior to classification.
when_to_use_negative:
- Input is not tabular data — the SHEEP architecture was built to deal with tabular
  data (e.g. radio fluxes/magnitudes are compatible, but non-tabular inputs are not)
- Your dataset differs from SDSS DR15+WISE and you have not re-optimised the models
  — the FLAML hyper-parameters are tuned for SDSS DR15+WISE and must be changed according
  to your dataset
- Redshift information is already available in the catalog, making a photo-z estimation
  stage unnecessary
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: SHEEP
  role: Pipeline hosting the chain-regressor photo-z estimation step that runs prior
    to classification
  repo: github.com/pedro-acunha/SHEEP
- name: XGBoost
  role: Gradient-boosted regressor combined in the chain-regressor ensemble to predict
    source redshifts
  repo: github.com/pedro-acunha/SHEEP
- name: LightGBM
  role: Gradient-boosted regressor combined in the chain-regressor ensemble to predict
    source redshifts
  repo: github.com/pedro-acunha/SHEEP
- name: CatBoost
  role: Gradient-boosted regressor combined in the chain-regressor ensemble to predict
    source redshifts
  repo: github.com/pedro-acunha/SHEEP
- name: FLAML
  role: Hyper-parameter optimisation of the regressors; re-run whenever changes are
    made in the input data
- name: Python
  role: Implementation language for photo_z.py and the SHEEP pipeline
  repo: github.com/pedro-acunha/SHEEP
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202243135
    title: Photometric redshift-aided classification using ensemble learning
  source_content_hash: sha256:030fca7fe2d1c5ecb69cf384e1486f49e7ad0558a799fce67d0d329a39021df5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CatBoost
  - FLAML
  - LightGBM
  - Python
  - SHEEP
  - XGBoost
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# chain-regressor-ensemble-modeling

## Summary

Predict per-source photometric redshifts in SHEEP using a chain-regressor that ensembles XGBoost, LightGBM, and CatBoost regressors over SDSS DR15+WISE photometry, so the redshift can be added as a feature that improves downstream galaxy/quasar/star classification.

## When to use

Use when a tabular astronomical catalog (e.g. SDSS+WISE photometry) lacks redshifts but the classification pipeline benefits from redshift as a feature; the photo-z must be estimated prior to classification.

## When NOT to use

- Input is not tabular data — the SHEEP architecture was built to deal with tabular data (e.g. radio fluxes/magnitudes are compatible, but non-tabular inputs are not)
- Your dataset differs from SDSS DR15+WISE and you have not re-optimised the models — the FLAML hyper-parameters are tuned for SDSS DR15+WISE and must be changed according to your dataset
- Redshift information is already available in the catalog, making a photo-z estimation stage unnecessary

## Inputs

- Feature-engineered SDSS DR15+WISE photometry table of ~3.5 million astronomical sources (produced by create_data.py from CasJobs SQL queries)
- FLAML-optimised hyper-parameters for the XGBoost/LightGBM/CatBoost regressors

## Outputs

- photo_z_predictions.csv with one photo-z estimate per catalog source
- Persisted fitted chain-regressor model reused as an additional feature by the downstream classification stage

## How to apply

Load the feature-engineered SDSS DR15+WISE photometry table produced by create_data.py (data retrieved via the sql_query_casjobs SQL queries from SDSS SkyServer CasJobs). Train the chain-regressor ensemble described in photo_z.py, combining gradient-boosted regressors from XGBoost, LightGBM, and CatBoost to predict source redshifts from the photometric features, with hyper-parameters obtained using FLAML. Apply the trained chain-regressor to every source in the catalog to generate per-source photo-z estimates, save them to photo_z_predictions.csv, and persist the fitted regressor for reuse as an additional feature by the classification stage (clf_multi.py or clf_1vsall.py). Re-run FLAML whenever changes are made in the input data, since the models are optimised for SDSS DR15+WISE.

## Related tools

- **SHEEP** (Pipeline hosting the chain-regressor photo-z estimation step that runs prior to classification) — github.com/pedro-acunha/SHEEP
- **XGBoost** (Gradient-boosted regressor combined in the chain-regressor ensemble to predict source redshifts) — github.com/pedro-acunha/SHEEP
- **LightGBM** (Gradient-boosted regressor combined in the chain-regressor ensemble to predict source redshifts) — github.com/pedro-acunha/SHEEP
- **CatBoost** (Gradient-boosted regressor combined in the chain-regressor ensemble to predict source redshifts) — github.com/pedro-acunha/SHEEP
- **FLAML** (Hyper-parameter optimisation of the regressors; re-run whenever changes are made in the input data)
- **Python** (Implementation language for photo_z.py and the SHEEP pipeline) — github.com/pedro-acunha/SHEEP

## Examples

```
python photo_z.py  # train the XGBoost/LightGBM/CatBoost chain-regressor with FLAML-tuned hyper-parameters and save photo_z_predictions.csv
```

## Evaluation signals

- photo_z_predictions.csv exists with exactly one photo-z estimate per catalog source
- Per-source photo-z predictions are complete and finite, consistent with the SDSS DR15+WISE input table
- The saved chain-regressor was fitted with FLAML-optimised hyper-parameters
- The photo-z feature is added to the dataset before classification model training, matching the SHEEP pipeline ordering
- Regression ensemble draws on all three learners (XGBoost, LightGBM, CatBoost) as combined in the chain-regressor approach

## Limitations

- The models are optimised for SDSS DR15+WISE; if used with a different dataset, the FLAML-optimised hyper-parameters must be changed accordingly
- No explicit numeric photo-z accuracy threshold is reported, so acceptance is qualitative (complete, finite per-source predictions)
- It is recommended to re-run FLAML every time changes are made in the input data

## Evidence

- [methods] photo-z via chain-regressor: "The chain-regressor approach used is described in the file "photo_z.py"."
- [abstract] photo-z as classification feature: "prior to performing the classification, SHEEP first estimates photometric redshifts, which are then placed into the data set as an additional feature"
- [abstract] ensemble learners: "combines the outputs from the XGBoost, LightGBM, and CatBoost learning algorithms to create stronger classifiers"
- [readme] dataset-specific hyper-parameters: "If you use it with a different dataset, please remember that the models are optimised for SDSS DR15+WISE. Do not forget to change them according to your dataset."
- [readme] FLAML re-run guidance: "The hyper-parameters obtained using FLAML are described. It is recommended to re-run FLAML everytime changes are made in the input data."
