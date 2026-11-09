---
name: gradient-boosted-tree-regression
description: Use when tabular photometric survey data (e.g. SDSS+WISE magnitudes/fluxes)
  lacks a redshift value that is needed as a regression target or downstream feature,
  and a per-source photo-z estimate must be produced by an ensemble of gradient-boosted
  regressors.
when_to_use_negative:
- Input is not tabular photometric data (SHEEP's architecture targets tabular data,
  allowing sparse data; SDSS/WISE-style photometry is the optimised case)
- Redshifts are already available per source, so no regression step is needed
- The task is the classification step itself (galaxy/QSO/star via clf_multi.py or
  clf_1vsall.py) rather than the upstream photo-z regression
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: XGBoost
  role: Gradient-boosted regressor whose output is one member of the chain-regressor
    ensemble predicting photometric redshifts
- name: LightGBM
  role: Gradient-boosted regressor whose output is one member of the chain-regressor
    ensemble predicting photometric redshifts
- name: CatBoost
  role: Gradient-boosted regressor whose output is one member of the chain-regressor
    ensemble predicting photometric redshifts
- name: FLAML
  role: Hyper-parameter optimisation of the regressors; should be re-run whenever
    the input data change
- name: SHEEP
  role: Pipeline whose photo_z.py file implements the chain-regressor photo-z estimation
    stage
  repo: github.com/pedro-acunha/SHEEP
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202243135
    title: Photometric redshift-aided classification using ensemble learning
  source_content_hash: sha256:e48d55a77773246eb9ce62f54be82e6fbdacc1abc06e1baceea1b164c3232b66
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# gradient-boosted-tree-regression

## Summary

Gradient-boosted tree regression as implemented in SHEEP's photometric-redshift estimation: an ensemble (chain-regressor) of XGBoost, LightGBM, and CatBoost regressors predicts per-source redshifts from SDSS DR15+WISE photometry, with hyper-parameters optimised via FLAML, before the redshift feature feeds downstream classification.

## When to use

Use when tabular photometric survey data (e.g. SDSS+WISE magnitudes/fluxes) lacks a redshift value that is needed as a regression target or downstream feature, and a per-source photo-z estimate must be produced by an ensemble of gradient-boosted regressors.

## When NOT to use

- Input is not tabular photometric data (SHEEP's architecture targets tabular data, allowing sparse data; SDSS/WISE-style photometry is the optimised case)
- Redshifts are already available per source, so no regression step is needed
- The task is the classification step itself (galaxy/QSO/star via clf_multi.py or clf_1vsall.py) rather than the upstream photo-z regression

## Inputs

- Feature-engineered SDSS DR15+WISE photometry table for astronomical sources (tabular, sparse-data tolerant), produced by create_data.py
- SQL data retrieval via 'sql_query_casjobs' queries on SDSS SkyServer CasJobs (https://skyserver.sdss.org/CasJobs/)

## Outputs

- photo_z_predictions.csv with one photometric-redshift estimate per catalog source
- Persisted fitted chain-regressor for reuse as an additional feature by the classification stage

## How to apply

Load the feature-engineered SDSS DR15+WISE photometry table produced by create_data.py (raw data obtainable via the sql_query_casjobs SQL queries on SDSS SkyServer CasJobs). Train the chain-regressor ensemble described in photo_z.py, combining gradient-boosted regressors from XGBoost, LightGBM, and CatBoost to predict source redshifts from the photometric features. Optimise regressor hyper-parameters with FLAML; the README recommends re-running FLAML every time the input data change, since the shipped models are optimised for SDSS DR15+WISE. Apply the fitted ensemble to every source in the catalog to generate per-source photo-z estimates, save them to photo_z_predictions.csv, and persist the fitted regressor so the redshift can be added as an additional feature for the downstream classification model training.

## Related tools

- **XGBoost** (Gradient-boosted regressor whose output is one member of the chain-regressor ensemble predicting photometric redshifts)
- **LightGBM** (Gradient-boosted regressor whose output is one member of the chain-regressor ensemble predicting photometric redshifts)
- **CatBoost** (Gradient-boosted regressor whose output is one member of the chain-regressor ensemble predicting photometric redshifts)
- **FLAML** (Hyper-parameter optimisation of the regressors; should be re-run whenever the input data change)
- **SHEEP** (Pipeline whose photo_z.py file implements the chain-regressor photo-z estimation stage) — github.com/pedro-acunha/SHEEP

## Examples

```
python photo_z.py
```

## Evaluation signals

- photo_z_predictions.csv exists and contains exactly one photo-z estimate per source in the SDSS DR15+WISE input table
- All per-source photo-z predictions are finite and qualitatively consistent with the SDSS DR15+WISE input table (no explicit numeric accuracy threshold is reported)
- The persisted regressor was fitted with FLAML-optimised hyper-parameters
- FLAML was re-run after any change to the input data, since the shipped models are optimised for SDSS DR15+WISE
- The redshift feature is actually injected into the classification training data, matching the paper's design of estimating photo-z prior to classification

## Limitations

- The shipped hyper-parameters are optimised for SDSS DR15+WISE; using a different dataset requires changing them (re-running FLAML)
- No explicit numeric photo-z accuracy threshold is reported, so acceptance of outputs is qualitative rather than threshold-based
- The paper's reported photo-z benefit is framed as an added feature for classification; the classification F1-scores (0.992 galaxies, 0.967 quasars, 0.985 stars) are classification metrics, not direct photo-z accuracy guarantees

## Evidence

- [abstract] chain-regressor photo-z with ensemble: "prior to performing the classification, SHEEP first estimates photometric redshifts, which are then placed into the data set as an additional feature"
- [abstract] ensemble members: "combines the outputs from the XGBoost, LightGBM, and CatBoost learning algorithms to create stronger classifiers"
- [readme] photo_z.py implementation file: "The chain-regressor approach used is described in the file "photo_z.py"."
- [readme] FLAML re-run caveat: "It is recommended to re-run FLAML everytime changes are made in the input data."
- [readme] dataset-specific optimisation: "If you use it with a different dataset, please remember that the models are optimised for SDSS DR15+WISE. Do not forget to change them according to your dataset."
