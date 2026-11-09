---
name: astronomical-follow-up-candidate-selection
description: Use when you have a large photometric source catalogue with per-source
  classification probabilities (e.g. 3.5 million SDSS+WISE sources scored by SHEEP)
  and need to flag sources with ambiguous class probabilities for follow-up observations.
when_to_use_negative:
- Input is not tabular data (SHEEP was built to deal with tabular data, e.g. SDSS/WISE
  photometry or radio fluxes/magnitudes, so non-tabular inputs are incompatible with
  the architecture)
- You are applying the models to a different dataset without re-running FLAML — the
  hyper-parameters are optimised for SDSS DR15+WISE only
- The goal is definitive classification with confident labels rather than surfacing
  ambiguous/uncertain sources for follow-up
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: SHEEP
  role: Pipeline whose trained classifiers (clf_multi.py, clf_1vsall.py) produce per-source
    galaxy/QSO/star class probabilities used to select uncertain sources for follow-up
  repo: github.com/pedro-acunha/SHEEP
- name: FLAML
  role: Provides the optimised hyper-parameters for the classification models; recommended
    to re-run whenever changes are made in the input data
- name: XGBoost
  role: Base learner whose outputs are combined into the ensemble classifiers that
    generate the per-source class probabilities
- name: LightGBM
  role: Base learner whose outputs are combined into the ensemble classifiers that
    generate the per-source class probabilities
- name: CatBoost
  role: Base learner whose outputs are combined into the ensemble classifiers that
    generate the per-source class probabilities
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1051/0004-6361/202243135
    title: Photometric redshift-aided classification using ensemble learning
  source_content_hash: sha256:d97d6e39cd266d46674dad7a194819fb52212f09ef7273b2a4f35e27d6647fe3
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
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# astronomical-follow-up-candidate-selection

## Summary

Identifies astronomical sources whose galaxy/QSO/star class probabilities from the SHEEP ensemble (XGBoost, LightGBM, CatBoost) are ambiguous or uncertain, ranking them as candidates for follow-up observations; feature importances provide supporting explainability.

## When to use

Use when you have a large photometric source catalogue with per-source classification probabilities (e.g. 3.5 million SDSS+WISE sources scored by SHEEP) and need to flag sources with ambiguous class probabilities for follow-up observations.

## When NOT to use

- Input is not tabular data (SHEEP was built to deal with tabular data, e.g. SDSS/WISE photometry or radio fluxes/magnitudes, so non-tabular inputs are incompatible with the architecture)
- You are applying the models to a different dataset without re-running FLAML — the hyper-parameters are optimised for SDSS DR15+WISE only
- The goal is definitive classification with confident labels rather than surfacing ambiguous/uncertain sources for follow-up

## Inputs

- Feature-engineered SDSS DR15+WISE photometric dataset (tabular, with photo-z estimates from photo_z.py as an additional classification feature)
- Trained SHEEP classification models from clf_multi.py (multi-class) and clf_1vsall.py (one-vs-all), with FLAML-optimised hyper-parameters
- 3.5-million-source catalogue of SDSS and WISE photometry

## Outputs

- uncertain_sources.csv — catalogue filtered to sources with ambiguous/uncertain class probabilities, ranked as follow-up observation candidates
- feature_importance.csv — aggregated, ranked feature-importance table from the XGBoost/LightGBM/CatBoost base learners

## How to apply

Load the feature-engineered SDSS DR15+WISE photometric dataset produced by 'create_data.py', ensuring the photometric-redshift estimates from the chain-regressor in 'photo_z.py' are present as the additional classification feature. Load the trained models from 'clf_multi.py' (multi-class) and 'clf_1vsall.py' (one-vs-all), using the hyper-parameters obtained with FLAML, which combine the outputs of XGBoost, LightGBM, and CatBoost learners. Apply the models to the 3.5-million-source catalogue to obtain per-source class probabilities for the galaxy, QSO, and star categories, then filter the catalogue to flag sources with ambiguous/uncertain probabilities and rank them as follow-up candidates in 'uncertain_sources.csv'. Optionally extract feature-importance scores from each base learner and aggregate them into a ranked table ('feature_importance.csv') to explain the classifications. Note that the models are optimised for SDSS DR15+WISE, so hyper-parameters should be re-optimised if the input data changes.

## Related tools

- **SHEEP** (Pipeline whose trained classifiers (clf_multi.py, clf_1vsall.py) produce per-source galaxy/QSO/star class probabilities used to select uncertain sources for follow-up) — github.com/pedro-acunha/SHEEP
- **FLAML** (Provides the optimised hyper-parameters for the classification models; recommended to re-run whenever changes are made in the input data)
- **XGBoost** (Base learner whose outputs are combined into the ensemble classifiers that generate the per-source class probabilities)
- **LightGBM** (Base learner whose outputs are combined into the ensemble classifiers that generate the per-source class probabilities)
- **CatBoost** (Base learner whose outputs are combined into the ensemble classifiers that generate the per-source class probabilities)

## Examples

```
python clf_multi.py
```

## Evaluation signals

- uncertain_sources.csv exists and is a well-formed CSV table containing sources flagged as ambiguous/uncertain and ranked as follow-up candidates
- feature_importance.csv exists and is a well-formed ranked table of aggregated feature-importance scores from the ensemble base learners
- Per-source output contains class probabilities for all three categories (galaxy, QSO, star)
- Classification behaviour remains consistent with the reported F1-scores: 0.992 (galaxies), 0.967 (quasars), 0.985 (stars)
- If the input dataset changed, FLAML was re-run and the hyper-parameters updated before applying the models

## Limitations

- The models are optimised for SDSS DR15+WISE; if used with a different dataset, hyper-parameters must be changed accordingly (re-run FLAML every time changes are made in the input data)
- Data must be tabular; other tabular data types (e.g. radio fluxes/magnitudes) are compatible, but non-tabular data is not
- The pipeline requires photometric redshifts estimated first (via the chain-regressor in photo_z.py) and added as a classification feature for the reported performance

## Evidence

- [abstract] The resulting F1 -scores are as follows: 0.992 for galaxies; 0.967 for quasars; and 0.985 for stars.: "The resulting F1 -scores are as follows: 0.992 for galaxies; 0.967 for quasars; and 0.985 for stars."
- [methods] the models are optimised for SDSS DR15+WISE.: "the models are optimised for SDSS DR15+WISE."
- [methods] Multi-class: see file "clf_multi.py".: "Multi-class: see file "clf_multi.py"."
- [readme] It is recommended to re-run FLAML everytime changes are made in the input data.: "It is recommended to re-run FLAML everytime changes are made in the input data."
- [readme] The code is ready to be used and reproduced the results in the paper.: "The code is ready to be used and reproduced the results in the paper."
