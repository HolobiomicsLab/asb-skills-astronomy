---
name: one-versus-all-source-classification
description: Use when classifying SDSS/WISE photometric sources into galaxies, quasars,
  and stars with per-class control, and when a multi-class model is not required —
  the one-vs-all path in SHEEP gives per-class F1-scores of 0.992/0.967/0.985.
when_to_use_negative:
- The input data is not tabular photometry compatible with SHEEP's architecture (e.g.
  non-tabular imagery) — the README states it was built for tabular data, allowing
  sparse data
- A single coherent multi-class model is required instead of per-class outputs — use
  SHEEP's multi-class methodology in clf_multi.py
- The dataset is not SDSS DR15+WISE and the FLAML-tuned hyper-parameters have not
  been re-optimised, since models are optimised for SDSS DR15+WISE
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: SHEEP
  role: Pipeline hosting the one-versus-all classification methodology with meta-learner
    correction
  repo: https://github.com/pedro-acunha/SHEEP
- name: XGBoost
  role: Base learner combined in the one-versus-all ensemble classifier
  repo: https://github.com/pedro-acunha/SHEEP
- name: LightGBM
  role: Base learner combined in the one-versus-all ensemble classifier
  repo: https://github.com/pedro-acunha/SHEEP
- name: CatBoost
  role: Base learner combined in the one-versus-all ensemble classifier
  repo: https://github.com/pedro-acunha/SHEEP
- name: FLAML
  role: Hyper-parameter optimisation of the ensemble learners; re-run whenever input
    data changes
  repo: https://github.com/pedro-acunha/SHEEP
- name: Python
  role: Implementation language for clf_1vsall.py and the evaluation code
  repo: https://github.com/pedro-acunha/SHEEP
- name: Jupyter Notebook
  role: Recommended environment for running and visualising the classification workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202243135
    title: Photometric redshift-aided classification using ensemble learning
  source_content_hash: sha256:d5bd753756bc9e79cc7d26f61fe861ad41483664b337653ae46ee672d53813ac
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
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# one-versus-all-source-classification

## Summary

Train a separate one-versus-all ensemble classifier (XGBoost + LightGBM + CatBoost) for each astronomical source class — galaxy, QSO, star — then reconcile the three per-class predictions with a meta-learner correction step, as implemented in SHEEP's clf_1vsall.py.

## When to use

Use when classifying SDSS/WISE photometric sources into galaxies, quasars, and stars with per-class control, and when a multi-class model is not required — the one-vs-all path in SHEEP gives per-class F1-scores of 0.992/0.967/0.985.

## When NOT to use

- The input data is not tabular photometry compatible with SHEEP's architecture (e.g. non-tabular imagery) — the README states it was built for tabular data, allowing sparse data
- A single coherent multi-class model is required instead of per-class outputs — use SHEEP's multi-class methodology in clf_multi.py
- The dataset is not SDSS DR15+WISE and the FLAML-tuned hyper-parameters have not been re-optimised, since models are optimised for SDSS DR15+WISE

## Inputs

- Feature-engineered SDSS DR15+WISE photometric table for ~3.5 million astronomical sources (from sql_query_casjobs + create_data.py)
- Chain-regressor photometric redshift estimates added as an additional classification feature
- FLAML-optimised hyper-parameters for XGBoost, LightGBM, and CatBoost base learners

## Outputs

- Corrected per-class one-versus-all predictions (galaxy, QSO, star) for every source
- Per-class F1-scores written to f1_scores_1vsall.csv (expected ~0.992 galaxies, ~0.967 quasars, ~0.985 stars)

## How to apply

Load the feature-engineered SDSS DR15+WISE photometric table (built from CasJobs SQL query output and create_data.py), augmented with chain-regressor photometric redshift estimates as an additional classification feature. For each class (galaxy, QSO, star), train a one-versus-all ensemble in clf_1vsall.py that combines XGBoost, LightGBM, and CatBoost base learners. Apply the FLAML-optimised hyper-parameters delivered with SHEEP, and re-run FLAML whenever the input data changes, as the README recommends. Pass the three per-class binary outputs through the meta-learner correction step implemented in clf_1vsall.py so the per-class predictions form a coherent assignment, then compute per-class F1-scores and write them to f1_scores_1vsall.csv as a distinct output from the multi-class methodology (clf_multi.py).

## Related tools

- **SHEEP** (Pipeline hosting the one-versus-all classification methodology with meta-learner correction) — https://github.com/pedro-acunha/SHEEP
- **XGBoost** (Base learner combined in the one-versus-all ensemble classifier) — https://github.com/pedro-acunha/SHEEP
- **LightGBM** (Base learner combined in the one-versus-all ensemble classifier) — https://github.com/pedro-acunha/SHEEP
- **CatBoost** (Base learner combined in the one-versus-all ensemble classifier) — https://github.com/pedro-acunha/SHEEP
- **FLAML** (Hyper-parameter optimisation of the ensemble learners; re-run whenever input data changes) — https://github.com/pedro-acunha/SHEEP
- **Python** (Implementation language for clf_1vsall.py and the evaluation code) — https://github.com/pedro-acunha/SHEEP
- **Jupyter Notebook** (Recommended environment for running and visualising the classification workflow)

## Examples

```
python clf_1vsall.py
```

## Evaluation signals

- Per-class F1-scores match the reported values — 0.992 for galaxies, 0.967 for quasars, 0.985 for stars — within a small tolerance on the optimised SDSS DR15+WISE setup
- f1_scores_1vsall.csv is produced and kept as a distinct output from the multi-class methodology (clf_multi.py)
- Each of the three one-versus-all classifiers (galaxy, QSO, star) trains successfully with the XGBoost, LightGBM, and CatBoost base learners combined
- Meta-learner correction is applied to the raw per-class predictions before evaluation, so corrected assignments are mutually coherent
- FLAML hyper-parameters were re-optimised on the current input data if any features were changed

## Limitations

- Models and FLAML hyper-parameters are optimised for SDSS DR15+WISE; using a different dataset requires re-running FLAML and adjusting hyper-parameters
- The photo-z feature comes from a chain-regressor estimate, so classification quality depends on photometric redshift accuracy
- One-versus-all yields three separate per-class predictions that must be reconciled by the meta-learner; class-probability conflicts are a failure mode if the correction step is skipped

## Evidence

- [abstract] SHEEP contains two distinct classification methodologies: (i) Multi-class and (ii) one versus all with correction by a meta-learner.: "SHEEP contains two distinct classification methodologies: (i) Multi-class and (ii) one versus all with correction by a meta-learner."
- [abstract] The resulting F1-scores are 0.992 for galaxies; 0.967 for quasars; and 0.985 for stars.: "The resulting F1 -scores are as follows: 0.992 for galaxies; 0.967 for quasars; and 0.985 for stars."
- [readme] One vs all: see file clf_1vsall.py.: "One vs all: see file "clf_1vsall.py"."
- [readme] It is recommended to re-run FLAML everytime changes are made in the input data.: "It is recommended to re-run FLAML everytime changes are made in the input data."
- [methods] The models are optimised for SDSS DR15+WISE.: "the models are optimised for SDSS DR15+WISE."
