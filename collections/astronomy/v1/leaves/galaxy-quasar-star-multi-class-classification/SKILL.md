---
name: galaxy-quasar-star-multi-class-classification
description: Use when the task is classifying tabular photometric catalogues (e.g.
  SDSS DR15+WISE magnitudes of millions of sources) into galaxy/QSO/star classes and
  per-class F1-scores around 0.96–0.99 are the target benchmark.
when_to_use_negative:
- Input is not tabular photometric data — the architecture is built for tabular data
  (e.g. SDSS/WISE photometry, radio fluxes/magnitudes), not images or spectra
- You are applying the pre-trained models unchanged to a dataset other than SDSS DR15+WISE
  without re-running FLAML — the README warns the models are optimised for SDSS DR15+WISE
  and hyper-parameters must be changed accordingly
- The goal is the one-versus-all classification variant — that is a separate methodology
  implemented in 'clf_1vsall.py' with meta-learner correction, not this multi-class
  file
edam_operation: http://edamontology.org/operation_3454
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_3360
tools:
- name: SHEEP
  role: Pipeline providing the multi-class classification methodology (clf_multi.py)
    for galaxy/QSO/star source classification
  repo: github.com/pedro-acunha/SHEEP
- name: XGBoost
  role: One of three base learners whose outputs are combined into the stronger classifier
  repo: github.com/pedro-acunha/SHEEP
- name: LightGBM
  role: One of three base learners whose outputs are combined into the stronger classifier
  repo: github.com/pedro-acunha/SHEEP
- name: CatBoost
  role: One of three base learners whose outputs are combined into the stronger classifier
  repo: github.com/pedro-acunha/SHEEP
- name: FLAML
  role: Hyper-parameter optimisation of the classifiers; must be re-run whenever the
    input data changes
  repo: github.com/pedro-acunha/SHEEP
- name: SDSS SkyServer CasJobs
  role: Source of the SDSS DR15+WISE photometric catalogue via the sql_query_casjobs
    SQL queries
- name: Python
  role: Implementation language, used with Jupyter Notebooks for visualisation and
    interactivity
  repo: github.com/pedro-acunha/SHEEP
- name: Jupyter Notebook
  role: Recommended environment for running and visualising the pipeline
  repo: github.com/pedro-acunha/SHEEP
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202243135
    title: Photometric redshift-aided classification using ensemble learning
  source_content_hash: sha256:69ff21487b3b785479d0ac99445ad0a1a7b9ee8e22b98c4447d3400791dd986a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CatBoost
  - FLAML
  - Jupyter Notebook
  - LightGBM
  - Python
  - SDSS SkyServer CasJobs
  - SHEEP
  - XGBoost
  edam_operation: http://edamontology.org/operation_3454
  edam_topics:
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# galaxy-quasar-star-multi-class-classification

## Summary

SHEEP multi-class methodology classifies SDSS+WISE astronomical sources into galaxies, quasars, and stars by combining XGBoost, LightGBM, and CatBoost outputs, with photometric redshifts estimated beforehand and appended as a classification feature.

## When to use

Use when the task is classifying tabular photometric catalogues (e.g. SDSS DR15+WISE magnitudes of millions of sources) into galaxy/QSO/star classes and per-class F1-scores around 0.96–0.99 are the target benchmark.

## When NOT to use

- Input is not tabular photometric data — the architecture is built for tabular data (e.g. SDSS/WISE photometry, radio fluxes/magnitudes), not images or spectra
- You are applying the pre-trained models unchanged to a dataset other than SDSS DR15+WISE without re-running FLAML — the README warns the models are optimised for SDSS DR15+WISE and hyper-parameters must be changed accordingly
- The goal is the one-versus-all classification variant — that is a separate methodology implemented in 'clf_1vsall.py' with meta-learner correction, not this multi-class file

## Inputs

- SDSS DR15+WISE photometry of ~3.5 million astronomical sources (tabular magnitudes/fluxes, sparse data allowed), retrieved via CasJobs SQL queries
- Photometric redshift values estimated per source by the chain-regressor (photo_z.py), appended as an extra feature

## Outputs

- Per-source galaxy/quasar/star class predictions
- Per-class F1-scores (benchmark: 0.992 galaxies, 0.967 quasars, 0.985 stars)

## How to apply

Retrieve the SDSS DR15+WISE photometric catalogue of ~3.5 million sources from SkyServer CasJobs using the SQL queries in 'sql_query_casjobs'. Engineer features on the tabular photometry following 'create_data.py'. Estimate photometric redshifts with the chain-regressor in 'photo_z.py' and place the photo-z values into the dataset as an additional classification feature — the paper reports this yields significant improvements in classification performance. Optimise hyper-parameters with FLAML, re-running FLAML whenever the input data changes, since the shipped hyper-parameters are optimised for SDSS DR15+WISE. Train the multi-class model in 'clf_multi.py', which combines XGBoost, LightGBM, and CatBoost outputs into a stronger classifier, then compute per-class F1-scores. Validate against the reported benchmarks: 0.992 (galaxies), 0.967 (quasars), 0.985 (stars).

## Related tools

- **SHEEP** (Pipeline providing the multi-class classification methodology (clf_multi.py) for galaxy/QSO/star source classification) — github.com/pedro-acunha/SHEEP
- **XGBoost** (One of three base learners whose outputs are combined into the stronger classifier) — github.com/pedro-acunha/SHEEP
- **LightGBM** (One of three base learners whose outputs are combined into the stronger classifier) — github.com/pedro-acunha/SHEEP
- **CatBoost** (One of three base learners whose outputs are combined into the stronger classifier) — github.com/pedro-acunha/SHEEP
- **FLAML** (Hyper-parameter optimisation of the classifiers; must be re-run whenever the input data changes) — github.com/pedro-acunha/SHEEP
- **SDSS SkyServer CasJobs** (Source of the SDSS DR15+WISE photometric catalogue via the sql_query_casjobs SQL queries)
- **Python** (Implementation language, used with Jupyter Notebooks for visualisation and interactivity) — github.com/pedro-acunha/SHEEP
- **Jupyter Notebook** (Recommended environment for running and visualising the pipeline) — github.com/pedro-acunha/SHEEP

## Examples

```
python clf_multi.py  # multi-class galaxy/QSO/star classification; re-run FLAML hyper-parameter search if the input SDSS DR15+WISE data changed
```

## Evaluation signals

- Reproduced per-class F1-scores match the paper benchmarks: 0.992 galaxies, 0.967 quasars, 0.985 stars on the 3.5-million-source SDSS+WISE set
- Photo-z feature is present in the classification input table (chain-regressor output appended before training)
- Hyper-parameter search re-ran with FLAML after any change to the input data, rather than reusing SDSS DR15+WISE-tuned values
- Classifier outputs combine all three base learners (XGBoost, LightGBM, CatBoost), not a single model
- Dataset size (~3.5 million sources) and feature set match the create_data.py feature-engineering output

## Limitations

- Models and hyper-parameters are optimised for SDSS DR15+WISE; using a different dataset requires re-running FLAML and adapting features
- Pipeline is designed for tabular data only; other data types must be tabular (e.g. radio fluxes/magnitudes) to be compatible
- Reported F1-scores (0.992/0.967/0.985) were obtained on an essentially identical SDSS+WISE dataset; performance on other surveys is not benchmarked here

## Evidence

- [abstract] combines the outputs from the XGBoost, LightGBM, and CatBoost learning algorithms to create stronger classifiers: "combines the outputs from the XGBoost, LightGBM, and CatBoost learning algorithms to create stronger classifiers"
- [abstract] F1-scores benchmarks: "The resulting F1 -scores are as follows: 0.992 for galaxies; 0.967 for quasars; and 0.985 for stars."
- [abstract] photo-z as feature: "SHEEP first estimates photometric redshifts, which are then placed into the data set as an additional feature for classification model training"
- [methods] FLAML re-run advice: "The hyper-parameters obtained using FLAML are described. It is recommended to re-run FLAML everytime changes are made in the input data."
- [readme] dataset optimisation caveat: "please remember that the models are optimised for SDSS DR15+WISE. Do not forget to change them according to your dataset."
- [readme] data retrieval: "To create the data you can use the SQL queries provided in the file "sql_query_casjobs". The data can be retrieved from https://skyserver.sdss.org/CasJobs/ ."
