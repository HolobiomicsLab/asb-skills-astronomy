---
name: photometric-redshift-regression
description: Use when you have SDSS DR15+WISE (or other tabular) photometry of astronomical
  sources and need per-source redshift estimates to feed as an additional feature
  into a galaxy/quasar/star classification pipeline.
when_to_use_negative:
- Your goal is classification alone with no redshift feature — the classification
  stage (clf_multi.py / clf_1vsall.py) is the relevant skill, not photo-z regression.
- Your data are already optimised models' target domain but you changed the input
  data without re-running FLAML — the README warns models are optimised for SDSS DR15+WISE
  and hyper-parameters must be re-optimised for a different dataset.
- Input is non-tabular data (e.g. images or spectra); the SHEEP architecture is built
  for tabular data with sparse values.
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: SHEEP
  role: Pipeline implementing the chain-regressor photo-z estimation step via photo_z.py
  repo: https://github.com/pedro-acunha/SHEEP
- name: XGBoost
  role: Gradient-boosted regressor component of the photo-z chain-regressor ensemble
- name: LightGBM
  role: Gradient-boosted regressor component of the photo-z chain-regressor ensemble
- name: CatBoost
  role: Gradient-boosted regressor component of the photo-z chain-regressor ensemble
- name: FLAML
  role: Hyper-parameter optimisation of the photo-z regressors; re-run whenever input
    data change
- name: Python
  role: Language for photo_z.py and the rest of the pipeline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202243135
    title: Photometric redshift-aided classification using ensemble learning
  source_content_hash: sha256:988f8b947147453f8062924229a5769741fe6c00fe5cd5f887519c1c61df8b38
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pedro-acunha/SHEEP
  tools:
  - CatBoost
  - FLAML
  - LightGBM
  - Python
  - SHEEP
  - XGBoost
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# photometric-redshift-regression

## Summary

Estimates per-source photometric redshifts using a chain-regressor ensemble (XGBoost, LightGBM, CatBoost) trained on SDSS DR15+WISE photometry, so redshifts can be added as an extra feature to improve downstream source classification.

## When to use

Use when you have SDSS DR15+WISE (or other tabular) photometry of astronomical sources and need per-source redshift estimates to feed as an additional feature into a galaxy/quasar/star classification pipeline.

## When NOT to use

- Your goal is classification alone with no redshift feature — the classification stage (clf_multi.py / clf_1vsall.py) is the relevant skill, not photo-z regression.
- Your data are already optimised models' target domain but you changed the input data without re-running FLAML — the README warns models are optimised for SDSS DR15+WISE and hyper-parameters must be re-optimised for a different dataset.
- Input is non-tabular data (e.g. images or spectra); the SHEEP architecture is built for tabular data with sparse values.

## Inputs

- Feature-engineered SDSS DR15+WISE photometry table (tabular magnitudes/fluxes from create_data.py; sparse data supported)
- Spectroscopic redshift labels for training the chain-regressor (from the CasJobs-retrieved SDSS DR15+WISE data)

## Outputs

- photo_z_predictions.csv with one photometric-redshift estimate per catalog source
- Persisted, FLAML-optimised fitted regressor for reuse by the downstream classification stage
- Redshift column added to the input data table as an additional classification feature

## How to apply

Load the feature-engineered SDSS DR15+WISE photometry table produced by create_data.py (data retrievable via the sql_query_casjobs SQL queries from SDSS SkyServer CasJobs). Implement the chain-regressor approach from photo_z.py, which combines gradient-boosted regressors from XGBoost, LightGBM, and CatBoost to predict source redshifts from the photometric features. Optimise regressor hyper-parameters with FLAML, and re-run FLAML whenever the input data change, since the shipped models are optimised for SDSS DR15+WISE. Apply the fitted chain-regressor to every source in the catalog, save the per-source estimates to photo_z_predictions.csv, and persist the fitted regressor so the downstream classification stage (clf_multi.py or clf_1vsall.py) can use photo-z as an additional feature — the paper reports this yields significant improvements in classification performance.

## Related tools

- **SHEEP** (Pipeline implementing the chain-regressor photo-z estimation step via photo_z.py) — https://github.com/pedro-acunha/SHEEP
- **XGBoost** (Gradient-boosted regressor component of the photo-z chain-regressor ensemble)
- **LightGBM** (Gradient-boosted regressor component of the photo-z chain-regressor ensemble)
- **CatBoost** (Gradient-boosted regressor component of the photo-z chain-regressor ensemble)
- **FLAML** (Hyper-parameter optimisation of the photo-z regressors; re-run whenever input data change)
- **Python** (Language for photo_z.py and the rest of the pipeline)

## Examples

```
python photo_z.py  # train FLAML-optimised chain-regressor (XGBoost+LightGBM+CatBoost) on SDSS DR15+WISE features; writes photo_z_predictions.csv
```

## Evaluation signals

- photo_z_predictions.csv exists and contains exactly one photo-z estimate per catalog source in the SDSS DR15+WISE input table
- All predicted redshifts are finite and consistent with the input photometry table (no missing per-source values)
- The persisted regressor was fitted with FLAML-optimised hyper-parameters, and FLAML was re-run if the input data changed
- Adding the photo-z feature to the classification stage yields improved three-class F1-scores relative to running classification without it (the paper reports significant improvements from adding photo-z)

## Limitations

- The shipped models are optimised for SDSS DR15+WISE; if used with a different dataset, FLAML hyper-parameter optimisation must be re-run and create_data.py feature engineering may need changes.
- No explicit numeric photo-z accuracy threshold is reported in the article, so acceptance is qualitative (complete, finite per-source predictions).
- The approach is designed for tabular photometric data; other data types are compatible only if tabular (e.g. radio fluxes/magnitudes).

## Evidence

- [readme] The chain-regressor approach used is described in the file "photo_z.py".: "The chain-regressor approach used is described in the file "photo_z.py"."
- [abstract] prior to performing the classification, SHEEP first estimates photometric redshifts, which are then placed into the data set as an additional feature: "prior to performing the classification, SHEEP first estimates photometric redshifts, which are then placed into the data set as an additional feature"
- [readme] The hyper-parameters obtained using FLAML are described. It is recommended to re-run FLAML everytime changes are made in the input data.: "The hyper-parameters obtained using FLAML are described. It is recommended to re-run FLAML everytime changes are made in the input data."
- [readme] To create the data you can use the SQL queries provided in the file "sql_query_casjobs". The data can be retrieved from https://skyserver.sdss.org/CasJobs/ .: "To create the data you can use the SQL queries provided in the file "sql_query_casjobs". The data can be retrieved from https://skyserver.sdss.org/CasJobs/ ."
- [abstract] combines the outputs from the XGBoost, LightGBM, and CatBoost learning algorithms to create stronger classifiers: "combines the outputs from the XGBoost, LightGBM, and CatBoost learning algorithms to create stronger classifiers"
