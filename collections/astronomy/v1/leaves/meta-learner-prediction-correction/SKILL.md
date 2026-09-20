---
name: meta-learner-prediction-correction
description: Use when classifying tabular astronomical sources (e.g., SDSS+WISE photometry)
  into galaxy/QSO/star classes one class at a time, and overlapping one-versus-all
  binary predictions need correction by a meta-learner.
when_to_use_negative:
- When a single multi-class model is preferred — SHEEP provides an alternative multi-class
  methodology in clf_multi.py, producing a separate output from the one-versus-all
  path
- When the input is not tabular data — the SHEEP architecture was built to deal with
  tabular data, so image or spectral inputs need conversion first
edam_operation: http://edamontology.org/operation_2990
tools:
- name: SHEEP
  role: Pipeline hosting the one-vs-all classification with meta-learner correction;
    the per-class logic is implemented in clf_1vsall.py
  repo: github.com/pedro-acunha/SHEEP
- name: XGBoost
  role: Base learner whose outputs are combined with LightGBM and CatBoost to create
    stronger per-class classifiers
- name: LightGBM
  role: Base learner combined with XGBoost and CatBoost in the one-versus-all ensemble
    for each source class
- name: CatBoost
  role: Base learner combined with XGBoost and LightGBM in the one-versus-all ensemble
    for each source class
- name: FLAML
  role: Supplies the optimised hyper-parameters for the base learners; recommended
    to be re-run every time changes are made to the input data
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202243135
    title: Photometric redshift-aided classification using ensemble learning
  source_content_hash: sha256:b802a9e9e12997ba908032a2eaba2fa3328df9491923f6948055f77c5de78835
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# meta-learner-prediction-correction

## Summary

SHEEP's one-versus-all classification methodology: train a per-class (galaxy, QSO, star) ensemble of XGBoost, LightGBM, and CatBoost base learners, then correct the per-class binary predictions with a meta-learner step. It is one of SHEEP's two distinct classification methodologies, complementing the multi-class approach.

## When to use

Use when classifying tabular astronomical sources (e.g., SDSS+WISE photometry) into galaxy/QSO/star classes one class at a time, and overlapping one-versus-all binary predictions need correction by a meta-learner.

## When NOT to use

- When a single multi-class model is preferred — SHEEP provides an alternative multi-class methodology in clf_multi.py, producing a separate output from the one-versus-all path
- When the input is not tabular data — the SHEEP architecture was built to deal with tabular data, so image or spectral inputs need conversion first

## Inputs

- Feature-engineered SDSS DR15+WISE photometric table of ~3.5 million astronomical sources (CasJobs SQL query output processed by create_data.py)
- Chain-regressor photometric redshift estimates added as an additional classification feature (photo_z.py output)
- FLAML-optimised hyper-parameters for XGBoost, LightGBM, and CatBoost base learners

## Outputs

- Corrected per-class one-versus-all predictions (galaxy / QSO / star) for every source
- Per-class F1-scores written to f1_scores_1vsall.csv (expected: 0.992 galaxies, 0.967 quasars, 0.985 stars)

## How to apply

Load the feature-engineered SDSS DR15+WISE photometric dataset (built via CasJobs SQL queries and create_data.py), augmented with chain-regressor photometric redshift estimates from photo_z.py as an additional classification feature. For each of the three classes (galaxy, QSO, star), train a one-versus-all ensemble classifier with the code in clf_1vsall.py, combining XGBoost, LightGBM, and CatBoost base learners. Apply the FLAML-optimised hyper-parameters, re-running FLAML on the current input data as the authors recommend whenever the input data changes. Correct the per-class one-versus-all predictions with the meta-learner step implemented in clf_1vsall.py, then compute per-class F1-scores and write them to f1_scores_1vsall.csv, kept distinct from the multi-class methodology output.

## Related tools

- **SHEEP** (Pipeline hosting the one-vs-all classification with meta-learner correction; the per-class logic is implemented in clf_1vsall.py) — github.com/pedro-acunha/SHEEP
- **XGBoost** (Base learner whose outputs are combined with LightGBM and CatBoost to create stronger per-class classifiers)
- **LightGBM** (Base learner combined with XGBoost and CatBoost in the one-versus-all ensemble for each source class)
- **CatBoost** (Base learner combined with XGBoost and LightGBM in the one-versus-all ensemble for each source class)
- **FLAML** (Supplies the optimised hyper-parameters for the base learners; recommended to be re-run every time changes are made to the input data)

## Examples

```
python clf_1vsall.py
```

## Evaluation signals

- Per-class F1-scores on the optimised SDSS DR15+WISE setup should match the reported values — 0.992 for galaxies, 0.967 for quasars, 0.985 for stars — within a small tolerance
- f1_scores_1vsall.csv is produced and kept as a distinct output from the multi-class methodology (clf_multi.py)
- Every source receives a meta-learner-corrected class assignment across the three one-versus-all classifiers, resolving overlaps between the galaxy, QSO, and star binary predictions
- FLAML was re-run on the current input data if any features or dataset composition changed, per the authors' recommendation

## Limitations

- Models are optimised for SDSS DR15+WISE; if the pipeline is used with a different dataset, the FLAML hyper-parameters must be re-run and updated accordingly
- The one-versus-all methodology requires the meta-learner correction step to reconcile per-class predictions; skipping it leaves conflicting per-class binary outputs unresolved
- The architecture accepts tabular data only (sparse data is allowed), so non-tabular data types cannot be used directly

## Evidence

- [abstract] SHEEP contains two distinct classification methodologies: (i) Multi-class and (ii) one versus all with correction by a meta-learner.: "SHEEP contains two distinct classification methodologies: (i) Multi-class and (ii) one versus all with correction by a meta-learner."
- [methods] clf_1vsall.py implements the one-vs-all classification: "One vs all: see file "clf_1vsall.py"."
- [abstract] XGBoost, LightGBM, and CatBoost outputs are combined: "combines the outputs from the XGBoost, LightGBM, and CatBoost learning algorithms to create stronger classifiers"
- [readme] FLAML hyper-parameters should be re-run on changed input data: "The hyper-parameters obtained using FLAML are described. It is recommended to re-run FLAML everytime changes are made in the input data."
- [abstract] Reported per-class F1-scores on 3.5 million sources: "The resulting F1 -scores are as follows: 0.992 for galaxies; 0.967 for quasars; and 0.985 for stars."
