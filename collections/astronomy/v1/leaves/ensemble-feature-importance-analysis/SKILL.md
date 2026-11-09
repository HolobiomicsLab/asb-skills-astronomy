---
name: ensemble-feature-importance-analysis
description: Use when a trained ensemble classifier's outputs must be explained (ranked
  feature importances) or when per-source class probabilities must be screened to
  isolate ambiguous astronomical sources for follow-up.
when_to_use_negative:
- Input is not tabular photometry (e.g. spectra or images) — the architecture is built
  for tabular data such as SDSS/WISE magnitudes or radio fluxes
- A different dataset is used without re-running FLAML — the models are optimised
  for SDSS DR15+WISE and hyper-parameters must be changed accordingly
- The goal is to retrain the classifier or tune hyper-parameters rather than explain
  an already-trained ensemble and select uncertain sources
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: SHEEP
  role: Pipeline providing the ensemble training, feature-importance explainability,
    and uncertain-source selection workflow
  repo: github.com/pedro-acunha/SHEEP
- name: XGBoost
  role: Base learner whose outputs and feature importances feed the ensemble aggregation
- name: LightGBM
  role: Base learner whose outputs and feature importances feed the ensemble aggregation
- name: CatBoost
  role: Base learner whose outputs and feature importances feed the ensemble aggregation
- name: FLAML
  role: Hyper-parameter optimisation for the base learners; re-run whenever the input
    data changes
- name: Python
  role: Implementation language of the SHEEP pipeline scripts
- name: Jupyter Notebook
  role: Recommended environment for visualisation and interactivity when inspecting
    feature importances
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1051/0004-6361/202243135
    title: Photometric redshift-aided classification using ensemble learning
  source_content_hash: sha256:cc5913c7cfe451104ac292f697de2db7d6c169d0df913bf46c470f6c6a89cce3
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# ensemble-feature-importance-analysis

## Summary

Extracts and aggregates feature-importance scores from the SHEEP ensemble (XGBoost + LightGBM + CatBoost) trained on SDSS DR15+WISE photometry of 3.5 million sources, and uses the resulting per-source class probabilities (galaxy/QSO/star) to flag uncertain classifications as candidates for follow-up observations. Provides model explainability and target-selection from the same trained classifiers.

## When to use

Use when a trained ensemble classifier's outputs must be explained (ranked feature importances) or when per-source class probabilities must be screened to isolate ambiguous astronomical sources for follow-up.

## When NOT to use

- Input is not tabular photometry (e.g. spectra or images) — the architecture is built for tabular data such as SDSS/WISE magnitudes or radio fluxes
- A different dataset is used without re-running FLAML — the models are optimised for SDSS DR15+WISE and hyper-parameters must be changed accordingly
- The goal is to retrain the classifier or tune hyper-parameters rather than explain an already-trained ensemble and select uncertain sources

## Inputs

- SDSS DR15+WISE photometric feature table of ~3.5 million astronomical sources (tabular, sparse-tolerant) produced by 'create_data.py'
- Photometric-redshift estimates from the chain-regressor in 'photo_z.py' used as an additional classification feature
- Trained ensemble classification models from 'clf_multi.py' and 'clf_1vsall.py' with FLAML-optimised hyper-parameters

## Outputs

- feature_importance.csv — ranked feature-importance table aggregated over the XGBoost, LightGBM, and CatBoost base learners
- uncertain_sources.csv — filtered catalogue of sources with ambiguous/uncertain galaxy, QSO, or star class probabilities, ranked as follow-up observation candidates

## How to apply

Load the feature-engineered SDSS DR15+WISE photometric table from 'create_data.py', ensuring the photometric-redshift column predicted by the chain-regressor in 'photo_z.py' is present, since adding photo-z as a classification feature yields significant performance improvements. Load the trained models from 'clf_multi.py' (multi-class) and 'clf_1vsall.py' (one-vs-all), whose hyper-parameters were obtained with FLAML and should be re-run whenever the input data changes. Extract feature-importance scores from each base learner (XGBoost, LightGBM, CatBoost), aggregate them into a single ranked table saved as feature_importance.csv. Apply the models to the 3.5-million-source catalogue to obtain per-source galaxy/QSO/star probabilities, then filter to flag ambiguous/uncertain classifications into uncertain_sources.csv ranked as follow-up candidates. Remember the models are optimised for SDSS DR15+WISE; adapt hyper-parameters if your dataset differs.

## Related tools

- **SHEEP** (Pipeline providing the ensemble training, feature-importance explainability, and uncertain-source selection workflow) — github.com/pedro-acunha/SHEEP
- **XGBoost** (Base learner whose outputs and feature importances feed the ensemble aggregation)
- **LightGBM** (Base learner whose outputs and feature importances feed the ensemble aggregation)
- **CatBoost** (Base learner whose outputs and feature importances feed the ensemble aggregation)
- **FLAML** (Hyper-parameter optimisation for the base learners; re-run whenever the input data changes)
- **Python** (Implementation language of the SHEEP pipeline scripts)
- **Jupyter Notebook** (Recommended environment for visualisation and interactivity when inspecting feature importances)

## Evaluation signals

- feature_importance.csv exists and is a well-formed CSV with a single ranked aggregation of base-learner importances
- uncertain_sources.csv exists and contains sources flagged with ambiguous class probabilities across galaxy/QSO/star categories
- Classification behaviour remains consistent with reported F1-scores: 0.992 for galaxies, 0.967 for quasars, and 0.985 for stars
- The photometric-redshift feature from 'photo_z.py' is present in the feature table, since omitting it degrades classification performance
- Importance aggregation covers all three base learners (XGBoost, LightGBM, CatBoost), not just a single model

## Limitations

- Models are optimised for SDSS DR15+WISE; using a different dataset requires changing the FLAML-optimised hyper-parameters
- Feature engineering (create_data.py) and photo-z estimation (photo_z.py) must precede this step; the additional photo-z feature is what drives the significant improvement
- It is recommended to re-run FLAML every time changes are made to the input data, otherwise importances and probabilities may reflect stale hyper-parameters

## Evidence

- [abstract] SHEEP, a new machine learning approach to the classic problem of astronomical source classification: "We present SHEEP, a new machine learning approach to the classic problem of astronomical source classification"
- [methods] hyper-parameters from FLAML: "The hyper-parameters obtained using FLAML are described. It is recommended to re-run FLAML everytime changes are made in the input data."
- [readme] feature engineering in create_data.py: "All the feature engineering process can be seen in the file "create_data.py". Make the necessary changes, creating and eliminating features, as needed for your project."
- [abstract] reported F1-scores: "The resulting F1 -scores are as follows: 0.992 for galaxies; 0.967 for quasars; and 0.985 for stars."
- [abstract] photo-z as additional classification feature: "SHEEP first estimates photometric redshifts, which are then placed into the data set as an additional feature for classification model training; this results in significant improvements"
