---
name: ensemble-learning-hyperparameter-optimization
description: Use when training gradient-boosting ensemble classifiers on tabular photometric
  (or other tabular) data where F1-score per class is the target metric and hyper-parameters
  must be re-tuned after any change to the input features.
when_to_use_negative:
- The input is not tabular data — SHEEP was built for tabular data (SDSS/WISE photometry,
  e.g. radio fluxes/magnitudes), not images or spectra
- Using a dataset other than SDSS DR15+WISE without re-running FLAML — the shipped
  hyper-parameters are optimised only for SDSS DR15+WISE
- You need probabilistic calibration or hierarchical taxonomy rather than flat galaxy/QSO/star
  labels — the pipeline's meta-learner corrects one-versus-all predictions but does
  not model class hierarchy
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: FLAML
  role: Hyper-parameter optimisation of the ensemble base learners; re-run whenever
    the input data changes
  repo: github.com/pedro-acunha/SHEEP
- name: XGBoost
  role: Base learner in the ensemble whose outputs are combined into stronger classifiers
  repo: github.com/pedro-acunha/SHEEP
- name: LightGBM
  role: Base learner in the ensemble whose outputs are combined into stronger classifiers
  repo: github.com/pedro-acunha/SHEEP
- name: CatBoost
  role: Base learner in the ensemble whose outputs are combined into stronger classifiers
  repo: github.com/pedro-acunha/SHEEP
- name: SHEEP
  role: Pipeline implementing the one-versus-all + meta-learner correction (clf_1vsall.py)
    and multi-class (clf_multi.py) methodologies
  repo: github.com/pedro-acunha/SHEEP
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202243135
    title: Photometric redshift-aided classification using ensemble learning
  source_content_hash: sha256:7330675004be5f9be0ab2fe2af132dc50f1b3cb0881f78c197b575b5e4b835c6
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
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# ensemble-learning-hyperparameter-optimization

## Summary

Optimise hyper-parameters for gradient-boosting ensembles (XGBoost, LightGBM, CatBoost) with FLAML, then combine them into one-versus-all or multi-class classifiers with a meta-learner correction, as done in SHEEP for SDSS DR15+WISE photometric source classification.

## When to use

Use when training gradient-boosting ensemble classifiers on tabular photometric (or other tabular) data where F1-score per class is the target metric and hyper-parameters must be re-tuned after any change to the input features.

## When NOT to use

- The input is not tabular data — SHEEP was built for tabular data (SDSS/WISE photometry, e.g. radio fluxes/magnitudes), not images or spectra
- Using a dataset other than SDSS DR15+WISE without re-running FLAML — the shipped hyper-parameters are optimised only for SDSS DR15+WISE
- You need probabilistic calibration or hierarchical taxonomy rather than flat galaxy/QSO/star labels — the pipeline's meta-learner corrects one-versus-all predictions but does not model class hierarchy

## Inputs

- Feature-engineered SDSS DR15+WISE photometric table of ~3.5 million astronomical sources (built via CasJobs SQL query output and create_data.py)
- Chain-regressor photometric redshift estimates added as an additional feature column
- Class labels: galaxy, QSO, star

## Outputs

- Per-class corrected one-versus-all predictions (galaxies, quasars, stars)
- Per-class F1-scores written to f1_scores_1vsall.csv (distinct from the multi-class methodology output)

## How to apply

Build the feature table with create_data.py (optionally augmenting it with chain-regressor photometric redshift estimates from photo_z.py, which the paper reports significantly improves classification). Train per-class one-versus-all classifiers in clf_1vsall.py (or multi-class in clf_multi.py), each combining XGBoost, LightGBM, and CatBoost base learners, then correct the per-class predictions with the meta-learner step in clf_1vsall.py. Run FLAML to obtain optimised hyper-parameters; the README explicitly recommends re-running FLAML every time the input data changes, and notes the shipped parameters are optimised for SDSS DR15+WISE and must be re-tuned for other datasets. Evaluate per-class F1-scores, expecting ~0.992 (galaxies), 0.967 (quasars), 0.985 (stars) on the 3.5-million-source SDSS+WISE set.

## Related tools

- **FLAML** (Hyper-parameter optimisation of the ensemble base learners; re-run whenever the input data changes) — github.com/pedro-acunha/SHEEP
- **XGBoost** (Base learner in the ensemble whose outputs are combined into stronger classifiers) — github.com/pedro-acunha/SHEEP
- **LightGBM** (Base learner in the ensemble whose outputs are combined into stronger classifiers) — github.com/pedro-acunha/SHEEP
- **CatBoost** (Base learner in the ensemble whose outputs are combined into stronger classifiers) — github.com/pedro-acunha/SHEEP
- **SHEEP** (Pipeline implementing the one-versus-all + meta-learner correction (clf_1vsall.py) and multi-class (clf_multi.py) methodologies) — github.com/pedro-acunha/SHEEP

## Examples

```
python clf_1vsall.py
```

## Evaluation signals

- Per-class F1-scores match the reported values (0.992 galaxies, 0.967 quasars, 0.985 stars) within a small tolerance on SDSS DR15+WISE
- f1_scores_1vsall.csv is produced as a distinct output from the multi-class methodology
- FLAML has been re-run after any change to the input features (e.g. adding photo-z as a feature)
- All three base learners (XGBoost, LightGBM, CatBoost) contribute to the ensemble, and the meta-learner correction step is applied to the one-versus-all predictions
- SHEEP's three-class F1-scores exceed the RandomForest-based baseline on the essentially identical data set

## Limitations

- Hyper-parameters are optimised for SDSS DR15+WISE; using a different dataset without re-running FLAML degrades performance
- The chain-regressor photo-z step must be applied first — the reported gains come from adding photometric redshifts as a classification feature
- The one-versus-all methodology and multi-class methodology are distinct; their outputs (e.g. f1_scores_1vsall.csv vs multi-class scores) should not be conflated

## Evidence

- [abstract] SHEEP contains two distinct classification methodologies: (i) Multi-class and (ii) one versus all with correction by a meta-learner.: "SHEEP contains two distinct classification methodologies: (i) Multi-class and (ii) one versus all with correction by a meta-learner."
- [methods] The hyper-parameters obtained using FLAML are described.: "The hyper-parameters obtained using FLAML are described. It is recommended to re-run FLAML everytime changes are made in the input data."
- [readme] If you use it with a different dataset, please remember that the models are optimised for SDSS DR15+WISE.: "If you use it with a different dataset, please remember that the models are optimised for SDSS DR15+WISE. Do not forget to change them according to your dataset."
- [readme] One vs all: see file "clf_1vsall.py".: "One vs all: see file "clf_1vsall.py"."
- [abstract] The resulting F1 -scores are as follows: 0.992 for galaxies; 0.967 for quasars; and 0.985 for stars.: "The resulting F1 -scores are as follows: 0.992 for galaxies; 0.967 for quasars; and 0.985 for stars."
