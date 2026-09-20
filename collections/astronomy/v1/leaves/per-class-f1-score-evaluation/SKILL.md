---
name: per-class-f1-score-evaluation
description: Use when a galaxy/QSO/star classifier (e.g. SHEEP's XGBoost/LightGBM/CatBoost
  ensemble) has been trained on SDSS DR15+WISE photometry and you need per-class F1-scores
  to validate against the reported values or to run baseline/ablation comparisons.
when_to_use_negative:
- Evaluating the photo-z chain-regressor itself (photo_z.py), which is a regression
  task, not a three-class F1 evaluation
- Data other than SDSS DR15+WISE photometry without first changing the FLAML-optimised
  hyper-parameters, since the models are optimised for SDSS DR15+WISE
- When only aggregate/overall accuracy is needed rather than the per-class galaxy/QSO/star
  breakdown the paper reports
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: SHEEP
  role: Ensemble classifier whose per-class predictions (galaxy/QSO/star) are scored
    with F1
  repo: github.com/pedro-acunha/SHEEP
- name: XGBoost
  role: Base learner whose combined outputs feed the classifier being evaluated
  repo: github.com/pedro-acunha/SHEEP
- name: LightGBM
  role: Base learner whose combined outputs feed the classifier being evaluated
  repo: github.com/pedro-acunha/SHEEP
- name: CatBoost
  role: Base learner whose combined outputs feed the classifier being evaluated
  repo: github.com/pedro-acunha/SHEEP
- name: FLAML
  role: Hyper-parameter optimisation that must be re-run before scoring whenever input
    data changes (e.g. photo-z ablation)
  repo: github.com/pedro-acunha/SHEEP
- name: RandomForest
  role: Recent baseline classifier whose per-class F1-scores are compared against
    SHEEP's
- name: Python
  role: Language used to compute F1-scores and write the comparison CSV tables
- name: Jupyter Notebook
  role: Environment for visualisation and interactivity during evaluation
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_006
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202243135
    title: Photometric redshift-aided classification using ensemble learning
  source_content_hash: sha256:74ccfa739a1925f971c936e078b5f7d241a0c95c91bda2e338a47400275db2d0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CatBoost
  - FLAML
  - Jupyter Notebook
  - LightGBM
  - Python
  - RandomForest
  - SHEEP
  - XGBoost
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# per-class-f1-score-evaluation

## Summary

Computes and reports per-class F1-scores for galaxies, quasars, and stars classified from SDSS+WISE photometry, so that SHEEP's performance (0.992 galaxies, 0.967 quasars, 0.985 stars) can be validated, compared against a RandomForest baseline, and used in photo-z ablation studies.

## When to use

Use when a galaxy/QSO/star classifier (e.g. SHEEP's XGBoost/LightGBM/CatBoost ensemble) has been trained on SDSS DR15+WISE photometry and you need per-class F1-scores to validate against the reported values or to run baseline/ablation comparisons.

## When NOT to use

- Evaluating the photo-z chain-regressor itself (photo_z.py), which is a regression task, not a three-class F1 evaluation
- Data other than SDSS DR15+WISE photometry without first changing the FLAML-optimised hyper-parameters, since the models are optimised for SDSS DR15+WISE
- When only aggregate/overall accuracy is needed rather than the per-class galaxy/QSO/star breakdown the paper reports

## Inputs

- Predicted and true class labels (galaxy, QSO, star) for the held-out test set of SDSS DR15+WISE sources
- Feature-engineered SDSS+WISE photometry (from create_data.py) optionally including chain-regressor photo-z estimates
- SHEEP ensemble predictions from clf_multi.py or clf_1vsall.py

## Outputs

- Per-class F1-scores for galaxies, quasars, and stars
- f1_scores_1vsall.csv (one-versus-all methodology, kept distinct from multi-class output)
- photo_z_ablation_f1_scores.csv (with-photo-z vs without-photo-z comparison)
- sheep_vs_randomforest_f1_comparison.csv (SHEEP vs RandomForest baseline)

## How to apply

After fitting the SHEEP ensemble (clf_multi.py for multi-class or clf_1vsall.py for one-versus-all with meta-learner correction) with FLAML-optimised hyper-parameters, generate predictions on the held-out test set and compute the F1-score separately for each of the three classes: galaxies, quasars, and stars. Keep the one-versus-all F1 output (f1_scores_1vsall.csv) distinct from the multi-class methodology output. For ablation work, recompute per-class F1-scores with the photo-z feature removed (re-running FLAML on the changed input data) and save the comparison as photo_z_ablation_f1_scores.csv; for baseline comparison, compute the RandomForest baseline's per-class F1 on essentially identical feature-engineered data and save the comparison as sheep_vs_randomforest_f1_comparison.csv. Validate by checking the with-photo-z scores are consistent with the reported 0.992 (galaxies), 0.967 (quasars), 0.985 (stars) on the 3.5 million source SDSS+WISE dataset.

## Related tools

- **SHEEP** (Ensemble classifier whose per-class predictions (galaxy/QSO/star) are scored with F1) — github.com/pedro-acunha/SHEEP
- **XGBoost** (Base learner whose combined outputs feed the classifier being evaluated) — github.com/pedro-acunha/SHEEP
- **LightGBM** (Base learner whose combined outputs feed the classifier being evaluated) — github.com/pedro-acunha/SHEEP
- **CatBoost** (Base learner whose combined outputs feed the classifier being evaluated) — github.com/pedro-acunha/SHEEP
- **FLAML** (Hyper-parameter optimisation that must be re-run before scoring whenever input data changes (e.g. photo-z ablation)) — github.com/pedro-acunha/SHEEP
- **RandomForest** (Recent baseline classifier whose per-class F1-scores are compared against SHEEP's)
- **Python** (Language used to compute F1-scores and write the comparison CSV tables)
- **Jupyter Notebook** (Environment for visualisation and interactivity during evaluation)

## Examples

```
pd.DataFrame({'class': ['galaxy', 'quasar', 'star'], 'f1': [0.992, 0.967, 0.985]}).to_csv('f1_scores_1vsall.csv', index=False)
```

## Evaluation signals

- With-photo-z per-class F1-scores on the 3.5 million source SDSS+WISE dataset are consistent with 0.992 (galaxies), 0.967 (quasars), 0.985 (stars) within a small tolerance
- The photo-z ablation shows a significant F1 improvement for the with-photo-z run over the without-photo-z run across the three classes
- SHEEP's per-class F1-scores exceed the RandomForest baseline's for all three classes on essentially identical data
- The one-versus-all F1 output (f1_scores_1vsall.csv) is written as a distinct file from the multi-class methodology output
- FLAML optimisation was re-run on the current input data before scoring, per the paper's recommendation

## Limitations

- The models are optimised for SDSS DR15+WISE; results on a different dataset will not match the reported F1-scores without retuning
- FLAML hyper-parameters must be re-run every time changes are made in the input data, otherwise F1 comparisons across configurations are invalid

## Evidence

- [abstract] Reported per-class F1 reference values: "The resulting F1 -scores are as follows: 0.992 for galaxies; 0.967 for quasars; and 0.985 for stars."
- [abstract] RandomForest baseline comparison target: "SHEEP is found to outperform a recent RandomForest-based classification approach using an essentially identical data set."
- [methods] Re-run FLAML before scoring changed inputs: "The hyper-parameters obtained using FLAML are described. It is recommended to re-run FLAML everytime changes are made in the input data."
- [readme] Models are dataset-optimised; scores may not transfer: "If you use it with a different dataset, please remember that the models are optimised for SDSS DR15+WISE. Do not forget to change them according to your dataset."
