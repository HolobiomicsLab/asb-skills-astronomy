---
name: photometric-feature-engineering
description: Use when building a source-classification or photo-z pipeline from raw
  SDSS/WISE (or other tabular) photometry downloaded via SkyServer CasJobs and you
  need engineered features before model training.
when_to_use_negative:
- Input is already a curated feature table optimised for SDSS DR15+WISE models — re-engineering
  features would require re-running FLAML hyper-parameter optimisation on the changed
  input data
- The data are not tabular — SHEEP's architecture was built specifically to deal with
  tabular data
- The goal is imaging-based classification rather than classification from photometric
  catalogues
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: SHEEP
  role: Pipeline providing create_data.py for feature engineering and photo_z.py for
    the chain-regressor photo-z estimation
  repo: github.com/pedro-acunha/SHEEP
- name: FLAML
  role: Hyper-parameter optimisation that must be re-run whenever features in the
    input data are changed
- name: SDSS SkyServer CasJobs
  role: Source of the raw SDSS+WISE photometric data via the provided SQL queries
  repo: https://skyserver.sdss.org/CasJobs/
- name: Python
  role: Implementation language for the feature-engineering and photo-z scripts
- name: Jupyter Notebook
  role: Recommended environment for visualisation and interactivity during feature
    engineering
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202243135
    title: Photometric redshift-aided classification using ensemble learning
  source_content_hash: sha256:c96515864ade6860f13d29590b5d17aeca400a1d5265563785fe11aeac26aeed
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# photometric-feature-engineering

## Summary

Feature engineering for tabular SDSS+WISE photometry of astronomical sources, implemented in SHEEP's create_data.py, to build the feature matrix (augmented with estimated photometric redshifts) used to train galaxy/QSO/star classifiers.

## When to use

Use when building a source-classification or photo-z pipeline from raw SDSS/WISE (or other tabular) photometry downloaded via SkyServer CasJobs and you need engineered features before model training.

## When NOT to use

- Input is already a curated feature table optimised for SDSS DR15+WISE models — re-engineering features would require re-running FLAML hyper-parameter optimisation on the changed input data
- The data are not tabular — SHEEP's architecture was built specifically to deal with tabular data
- The goal is imaging-based classification rather than classification from photometric catalogues

## Inputs

- SDSS DR15+WISE photometry of 3.5 million astronomical sources (retrieved via SDSS SkyServer CasJobs with the sql_query_casjobs SQL queries)
- Raw tabular photometric catalogue (magnitudes/fluxes), including sparse data, from other wavelength regimes (e.g. radio fluxes/magnitudes)

## Outputs

- Engineered feature table ready for classification (created/modified by create_data.py)
- Feature table augmented with a photometric-redshift column estimated by the chain-regressor (photo_z.py)

## How to apply

Retrieve the SDSS DR15+WISE photometric catalogue using the SQL queries in 'sql_query_casjobs' from https://skyserver.sdss.org/CasJobs/. Then apply the feature-engineering procedure in SHEEP's 'create_data.py', creating and eliminating features as needed for the project. Estimate photometric redshifts with the chain-regressor approach in 'photo_z.py' and append the photo-z values to the dataset as an additional classification feature — the paper reports this yields significant performance improvements. Because the shipped models are optimised for SDSS DR15+WISE, re-run FLAML hyper-parameter optimisation whenever the input features change.

## Related tools

- **SHEEP** (Pipeline providing create_data.py for feature engineering and photo_z.py for the chain-regressor photo-z estimation) — github.com/pedro-acunha/SHEEP
- **FLAML** (Hyper-parameter optimisation that must be re-run whenever features in the input data are changed)
- **SDSS SkyServer CasJobs** (Source of the raw SDSS+WISE photometric data via the provided SQL queries) — https://skyserver.sdss.org/CasJobs/
- **Python** (Implementation language for the feature-engineering and photo-z scripts)
- **Jupyter Notebook** (Recommended environment for visualisation and interactivity during feature engineering)

## Examples

```
python create_data.py  # feature engineering on the CasJobs-retrieved SDSS DR15+WISE table; then python photo_z.py to append photo-z features
```

## Evaluation signals

- Downstream multi-class classifier reaches per-class F1-scores near the reported benchmarks: 0.992 (galaxies), 0.967 (quasars), 0.985 (stars) on the 3.5M-source SDSS+WISE set
- Adding the photo-z feature yields a measurable improvement in classification performance relative to the same features without it
- FLAML hyper-parameters have been re-optimised after any change to the feature set, as recommended by the README
- The feature table matches the schema produced by create_data.py for SDSS DR15+WISE (models were optimised for this configuration)

## Limitations

- Models are optimised for SDSS DR15+WISE; if used with a different dataset, hyper-parameters must be changed and FLAML re-run
- The feature engineering is specific to tabular photometry; other tabular data (e.g. radio fluxes/magnitudes) are compatible, but non-tabular data are not
- Sparse data are allowed but may require feature handling decisions (creating/eliminating features) in create_data.py

## Evidence

- [methods] feature engineering in create_data.py: "All the feature engineering process can be seen in the file "create_data.py"."
- [abstract] photo-z as an added feature: "SHEEP first estimates photometric redshifts, which are then placed into the data set as an additional feature"
- [readme] re-run FLAML after input changes: "The hyper-parameters obtained using FLAML are described. It is recommended to re-run FLAML everytime changes are made in the input data."
- [readme] data retrieval via CasJobs: "To create the data you can use the SQL queries provided in the file "sql_query_casjobs". The data can be retrieved from https://skyserver.sdss.org/CasJobs/ ."
- [readme] models optimised for SDSS DR15+WISE: "If you use it with a different dataset, please remember that the models are optimised for SDSS DR15+WISE. Do not forget to change them according to your dataset."
