---
name: photometric-redshift-feature-ablation
description: Use when you need to attribute SHEEP's classification performance (F1-scores
  on SDSS+WISE sources) to the photometric-redshift feature by contrasting a with-photo-z
  run against a without-photo-z run.
when_to_use_negative:
- Your goal is a single best classifier, not a performance attribution — use the standard
  SHEEP multi-class (clf_multi.py) or one-vs-all (clf_1vsall.py) pipeline directly
  with the photo-z feature included.
- Your input data is not tabular photometry or uses a different survey — the models
  are optimised for SDSS DR15+WISE and reported F1 targets will not transfer.
- The dataset already lacks or cannot produce photometric-redshift estimates, so the
  with/without-photo-z contrast is meaningless.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3489
tools:
- name: SHEEP
  role: The classification pipeline whose photo-z feature contribution is being ablated;
    classifier code lives in clf_multi.py and photo-z estimation in photo_z.py.
  repo: github.com/pedro-acunha/SHEEP
- name: XGBoost
  role: One of the three base learners whose combined outputs form the SHEEP ensemble
    evaluated in both ablation arms.
  repo: github.com/pedro-acunha/SHEEP
- name: LightGBM
  role: Second base learner of the SHEEP ensemble, used identically in with-photo-z
    and without-photo-z runs.
  repo: github.com/pedro-acunha/SHEEP
- name: CatBoost
  role: Third base learner of the SHEEP ensemble, used identically in both ablation
    arms.
  repo: github.com/pedro-acunha/SHEEP
- name: FLAML
  role: Hyper-parameter optimisation; the README recommends re-running FLAML every
    time changes are made in the input data, i.e. when the photo-z column is removed.
  repo: github.com/pedro-acunha/SHEEP
- name: Python
  role: Implementation language for running the ablation pipeline (photo_z.py, clf_multi.py,
    create_data.py).
- name: Jupyter Notebook
  role: Recommended environment for visualisation and interactivity while running
    the ablation comparison.
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202243135
    title: Photometric redshift-aided classification using ensemble learning
  source_content_hash: sha256:c54abdd531d701b525c444e6524ef91b39853eb5e0d9d33c722cd84e6e0e09e1
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3489
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# photometric-redshift-feature-ablation

## Summary

Ablation skill for quantifying how much SHEEP's galaxy/quasar/star classification gains come from adding chain-regressor-estimated photometric redshifts as an extra feature, by re-running the same XGBoost/LightGBM/CatBoost ensemble pipeline with and without the photo-z column and comparing per-class F1-scores.

## When to use

Use when you need to attribute SHEEP's classification performance (F1-scores on SDSS+WISE sources) to the photometric-redshift feature by contrasting a with-photo-z run against a without-photo-z run.

## When NOT to use

- Your goal is a single best classifier, not a performance attribution — use the standard SHEEP multi-class (clf_multi.py) or one-vs-all (clf_1vsall.py) pipeline directly with the photo-z feature included.
- Your input data is not tabular photometry or uses a different survey — the models are optimised for SDSS DR15+WISE and reported F1 targets will not transfer.
- The dataset already lacks or cannot produce photometric-redshift estimates, so the with/without-photo-z contrast is meaningless.

## Inputs

- SDSS DR15+WISE photometric dataset of ~3.5 million sources (SQL query output from SkyServer CasJobs)
- Feature-engineered table from create_data.py including the chain-regressor photometric-redshift column from photo_z.py
- FLAML-optimised hyper-parameters (re-run per input-data change)

## Outputs

- Per-class F1-scores for the with-photo-z run (target: 0.992 galaxies, 0.967 quasars, 0.985 stars)
- Per-class F1-scores for the without-photo-z ablated run
- Ablation comparison table, e.g. photo_z_ablation_f1_scores.csv

## How to apply

Load the SDSS DR15+WISE photometric dataset (~3.5 million sources retrieved via the sql_query_casjobs SQL queries from SkyServer CasJobs, feature-engineered with create_data.py) and train the multi-class ensemble in clf_multi.py (XGBoost + LightGBM + CatBoost) using the full feature set that includes the photo-z column produced by the photo_z.py chain-regressor. Re-run the identical pipeline with the photo-z column removed, and per the README's recommendation re-run FLAML hyper-parameter optimisation each time the input data changes. Compute per-class F1-scores (galaxies, quasars, stars) for both runs and save the comparison table. The with-photo-z run should reproduce the reported scores of 0.992 (galaxies), 0.967 (quasars), and 0.985 (stars), and show a significant improvement over the ablated run.

## Related tools

- **SHEEP** (The classification pipeline whose photo-z feature contribution is being ablated; classifier code lives in clf_multi.py and photo-z estimation in photo_z.py.) — github.com/pedro-acunha/SHEEP
- **XGBoost** (One of the three base learners whose combined outputs form the SHEEP ensemble evaluated in both ablation arms.) — github.com/pedro-acunha/SHEEP
- **LightGBM** (Second base learner of the SHEEP ensemble, used identically in with-photo-z and without-photo-z runs.) — github.com/pedro-acunha/SHEEP
- **CatBoost** (Third base learner of the SHEEP ensemble, used identically in both ablation arms.) — github.com/pedro-acunha/SHEEP
- **FLAML** (Hyper-parameter optimisation; the README recommends re-running FLAML every time changes are made in the input data, i.e. when the photo-z column is removed.) — github.com/pedro-acunha/SHEEP
- **Python** (Implementation language for running the ablation pipeline (photo_z.py, clf_multi.py, create_data.py).)
- **Jupyter Notebook** (Recommended environment for visualisation and interactivity while running the ablation comparison.)

## Examples

```
python clf_multi.py  # run 1: full features incl. photo-z column from photo_z.py; then drop the photo-z column, re-run FLAML, and re-run clf_multi.py to get the ablated F1-scores
```

## Evaluation signals

- The with-photo-z run's per-class F1-scores match the reported 0.992 (galaxies), 0.967 (quasars), and 0.985 (stars) within a small tolerance.
- The with-photo-z run shows a significant F1 improvement over the without-photo-z run for all three classes.
- Both ablation arms use identical base learners and data splits; only the photo-z feature column and the re-run FLAML hyper-parameters differ.
- The ablation comparison table (e.g. photo_z_ablation_f1_scores.csv) contains both F1 sets and is reproducible from the same SDSS DR15+WISE input.

## Limitations

- The models and hyper-parameters are optimised for SDSS DR15+WISE; using a different dataset requires changing them, so reported F1 targets and photo-z gains may not transfer.
- FLAML hyper-parameters must be re-run when the input data changes (i.e. when removing the photo-z column); reusing the with-photo-z hyper-parameters for the ablated run would bias the comparison.
- The photo-z feature is produced by a chain-regressor (photo_z.py); errors in the photo-z estimates propagate into the classification feature.

## Evidence

- [abstract] Photo-z feature addition finding: "prior to performing the classification, SHEEP first estimates photometric redshifts, which are then placed into the data set as an additional feature"
- [abstract] Reported F1 targets: "The resulting F1 -scores are as follows: 0.992 for galaxies; 0.967 for quasars; and 0.985 for stars."
- [readme] FLAML re-run requirement: "The hyper-parameters obtained using FLAML are described. It is recommended to re-run FLAML everytime changes are made in the input data."
- [readme] Dataset scope caveat: "If you use it with a different dataset, please remember that the models are optimised for SDSS DR15+WISE. Do not forget to change them according to your dataset."
- [methods] Photo-z implementation location: "The chain-regressor approach used is described in the file "photo_z.py"."
