---
name: photometric-source-classification-uncertainty-flagging
description: Use when a photometric source catalogue needs class probabilities plus
  a ranked list of ambiguous/uncertain classifications for follow-up observation selection,
  and when feature importances are needed to explain the classifier.
when_to_use_negative:
- The input data is not tabular photometry (the SHEEP architecture was built for tabular
  data such as SDSS/WISE photometry, though other tabular data e.g. radio fluxes/magnitudes
  are compatible)
- The dataset differs substantially from SDSS DR15+WISE and you have not re-optimised
  the FLAML hyper-parameters for it
- The goal is confident source cataloguing rather than selecting uncertain/ambiguous
  classifications for follow-up
edam_operation: http://edamontology.org/operation_2990
tools:
- name: SHEEP
  role: Pipeline providing the multi-class and one-vs-all classification methodologies,
    feature-importance explainability, and uncertain-source selection
  repo: https://github.com/pedro-acunha/SHEEP
- name: XGBoost
  role: Base learner whose outputs are combined into the ensemble classifier
  repo: https://github.com/pedro-acunha/SHEEP
- name: LightGBM
  role: Base learner whose outputs are combined into the ensemble classifier
  repo: https://github.com/pedro-acunha/SHEEP
- name: CatBoost
  role: Base learner whose outputs are combined into the ensemble classifier
  repo: https://github.com/pedro-acunha/SHEEP
- name: FLAML
  role: Hyper-parameter optimisation for the classifiers; hyper-parameters should
    be re-run when input data changes
  repo: https://github.com/pedro-acunha/SHEEP
- name: Jupyter Notebook
  role: Recommended environment for running and visualising the pipeline
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1051/0004-6361/202243135
    title: Photometric redshift-aided classification using ensemble learning
  source_content_hash: sha256:13bbc35ed3ec94210a9f860b293f8ccec04801796bc59534bbe89034f705f320
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# photometric-source-classification-uncertainty-flagging

## Summary

Applies the SHEEP ensemble classifier (XGBoost + LightGBM + CatBoost, hyper-parameters from FLAML) to SDSS DR15+WISE photometry of ~3.5 million sources, then extracts feature-importance tables and flags sources with ambiguous galaxy/QSO/star class probabilities as follow-up candidates.

## When to use

Use when a photometric source catalogue needs class probabilities plus a ranked list of ambiguous/uncertain classifications for follow-up observation selection, and when feature importances are needed to explain the classifier.

## When NOT to use

- The input data is not tabular photometry (the SHEEP architecture was built for tabular data such as SDSS/WISE photometry, though other tabular data e.g. radio fluxes/magnitudes are compatible)
- The dataset differs substantially from SDSS DR15+WISE and you have not re-optimised the FLAML hyper-parameters for it
- The goal is confident source cataloguing rather than selecting uncertain/ambiguous classifications for follow-up

## Inputs

- Feature-engineered SDSS DR15+WISE photometric dataset (~3.5 million sources) with photo-z estimates from the chain-regressor included as a feature
- Trained SHEEP classification models (multi-class and one-vs-all) with FLAML hyper-parameters

## Outputs

- feature_importance.csv — aggregated, ranked feature-importance table across the XGBoost/LightGBM/CatBoost base learners
- uncertain_sources.csv — ranked list of sources with ambiguous galaxy/QSO/star class probabilities, flagged for follow-up observations

## How to apply

Load the feature-engineered SDSS DR15+WISE photometric table produced by 'create_data.py', making sure the photometric-redshift estimates from the chain-regressor in 'photo_z.py' are included as an additional classification feature (this improves classification performance). Run the multi-class classifier ('clf_multi.py') and/or the one-vs-all classifier with meta-learner correction ('clf_1vsall.py'), combining the outputs of the XGBoost, LightGBM, and CatBoost learners with the FLAML-derived hyper-parameters; re-run FLAML if the input data changed. Apply the trained models to the 3.5-million-source catalogue to obtain per-source probabilities for the galaxy, QSO, and star categories. Aggregate feature-importance scores from the ensemble's base learners into feature_importance.csv, and filter the catalogue to flag ambiguous class probabilities, ranking them as follow-up candidates in uncertain_sources.csv.

## Related tools

- **SHEEP** (Pipeline providing the multi-class and one-vs-all classification methodologies, feature-importance explainability, and uncertain-source selection) — https://github.com/pedro-acunha/SHEEP
- **XGBoost** (Base learner whose outputs are combined into the ensemble classifier) — https://github.com/pedro-acunha/SHEEP
- **LightGBM** (Base learner whose outputs are combined into the ensemble classifier) — https://github.com/pedro-acunha/SHEEP
- **CatBoost** (Base learner whose outputs are combined into the ensemble classifier) — https://github.com/pedro-acunha/SHEEP
- **FLAML** (Hyper-parameter optimisation for the classifiers; hyper-parameters should be re-run when input data changes) — https://github.com/pedro-acunha/SHEEP
- **Jupyter Notebook** (Recommended environment for running and visualising the pipeline)

## Examples

```
python photo_z.py && python clf_multi.py  # then apply models to the 3.5M-source catalogue and export feature_importance.csv and uncertain_sources.csv
```

## Evaluation signals

- feature_importance.csv and uncertain_sources.csv exist and are well-formed CSV tables
- Classification behaviour is consistent with the reported F1-scores: 0.992 for galaxies, 0.967 for quasars, and 0.985 for stars
- Per-source probabilities for the three classes (galaxy, QSO, star) are present for all ~3.5 million catalogue sources
- The photo-z feature from 'photo_z.py' was included as a classification feature — omitting it degrades performance significantly
- Uncertain sources are ranked, not just binary-flagged, so they can be prioritised for follow-up observations

## Limitations

- Models are optimised for SDSS DR15+WISE; the README warns to change hyper-parameters when using a different dataset, and to re-run FLAML every time changes are made in the input data
- The context does not specify a concrete probability threshold for 'uncertain'; the ambiguity flagging criterion must be chosen by the practitioner
- Feature importances and uncertain-source selection depend on the ensemble (XGBoost/LightGBM/CatBoost) outputs and are specific to the galaxy/QSO/star three-class problem

## Evidence

- [abstract] SHEEP first estimates photometric redshifts, which are then placed into the data set as an additional feature for classification model training: "SHEEP first estimates photometric redshifts, which are then placed into the data set as an additional feature for classification model training; this results in significant improvements in the"
- [abstract] ensemble base learners: "combines the outputs from the XGBoost, LightGBM, and CatBoost learning algorithms to create stronger classifiers"
- [methods] hyper-parameter re-optimisation: "The hyper-parameters obtained using FLAML are described. It is recommended to re-run FLAML everytime changes are made in the input data."
- [readme] dataset applicability: "If you use it with a different dataset, please remember that the models are optimised for SDSS DR15+WISE. Do not forget to change them according to your dataset."
- [readme] tabular data scope: "It was built to deal with tabular data, while allowing the use of sparse data."
