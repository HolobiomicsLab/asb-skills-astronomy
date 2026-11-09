---
name: photo-z-augmented-feature-handling
description: 'Use when classifying astronomical sources from SDSS and WISE tabular
  photometry into galaxies, QSOs and stars, and the input table lacks a distance/redshift
  feature: estimate photo-z first and append it as a classification feature.'
when_to_use_negative:
- Your input table already contains spectroscopic redshifts or a reliable distance/redshift
  column — the extra photo-z feature adds estimation noise rather than information
- Your data is not tabular photometry (e.g. images, spectra, time series) — SHEEP's
  architecture is built to deal with tabular data only, though other tabular photometric
  bands (e.g. radio fluxes/magnitudes) are compatible
- You are doing a standalone photo-z study rather than classification — here photo-z
  is only an intermediate feature for the classifier, not the final product
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: SHEEP
  role: Pipeline hosting the chain-regressor photo-z estimation stage that produces
    the augmented classification feature
  repo: https://github.com/pedro-acunha/SHEEP
- name: Python
  role: Implementation language for photo_z.py and the photo-z feature augmentation
    step
- name: Jupyter Notebook
  role: Interactive environment recommended for running and visualising the photo-z
    feature-engineering workflow
- name: FLAML
  role: Re-optimises classifier hyper-parameters after the photo-z feature changes
    the input data
- name: SDSS SkyServer CasJobs
  role: Source of the SDSS+WISE photometry used as input to photo-z estimation, via
    the provided SQL queries
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1051/0004-6361/202243135
    title: Photometric redshift-aided classification using ensemble learning
  source_content_hash: sha256:021797a449a23e1af8df5f0e8a932c9a077e364492e4c632cdf3ed0ce134bc63
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
  - SDSS SkyServer CasJobs
  - SHEEP
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# photo-z-augmented-feature-handling

## Summary

Augment a tabular astronomical source catalogue with photometric redshift estimates — produced by SHEEP's chain-regressor (photo_z.py) on SDSS DR15+WISE photometry — before classification, since adding photo-z as an extra feature yields significant gains in galaxy/QSO/star classification performance.

## When to use

Use when classifying astronomical sources from SDSS and WISE tabular photometry into galaxies, QSOs and stars, and the input table lacks a distance/redshift feature: estimate photo-z first and append it as a classification feature.

## When NOT to use

- Your input table already contains spectroscopic redshifts or a reliable distance/redshift column — the extra photo-z feature adds estimation noise rather than information
- Your data is not tabular photometry (e.g. images, spectra, time series) — SHEEP's architecture is built to deal with tabular data only, though other tabular photometric bands (e.g. radio fluxes/magnitudes) are compatible
- You are doing a standalone photo-z study rather than classification — here photo-z is only an intermediate feature for the classifier, not the final product

## Inputs

- SDSS DR15+WISE photometric data table of ~3.5 million astronomical sources (tabular, sparse-tolerant), retrievable via sql_query_casjobs from SDSS SkyServer CasJobs
- Feature-engineered dataset produced by create_data.py
- photo_z.py chain-regressor configuration from the SHEEP repository

## Outputs

- Per-source photometric-redshift estimates appended as an additional feature column in the classification input table
- Feature table consumed downstream by clf_multi.py (multi-class) and clf_1vsall.py (one-vs-all) classifiers

## How to apply

Retrieve the SDSS+WISE photometric catalogue via the provided CasJobs SQL queries (https://skyserver.sdss.org/CasJobs/) and run SHEEP's feature engineering in create_data.py, keeping sparse/missing tabular values. Then use the chain-regressor approach in photo_z.py to predict a photometric redshift per source, and place these photo-z estimates into the dataset as an additional feature column prior to classification-model training (clf_multi.py / clf_1vsall.py). The rationale is that photo-z carries distance information that pure photometric colours cannot, which 'results in significant improvements in the' classification performance. If you modify the input data (including the added photo-z feature), re-run the FLAML hyper-parameter optimisation since the models are optimised for SDSS DR15+WISE.

## Related tools

- **SHEEP** (Pipeline hosting the chain-regressor photo-z estimation stage that produces the augmented classification feature) — https://github.com/pedro-acunha/SHEEP
- **Python** (Implementation language for photo_z.py and the photo-z feature augmentation step)
- **Jupyter Notebook** (Interactive environment recommended for running and visualising the photo-z feature-engineering workflow)
- **FLAML** (Re-optimises classifier hyper-parameters after the photo-z feature changes the input data)
- **SDSS SkyServer CasJobs** (Source of the SDSS+WISE photometry used as input to photo-z estimation, via the provided SQL queries)

## Examples

```
python photo_z.py  # chain-regressor photo-z on the SDSS DR15+WISE table from create_data.py; append output as a feature before clf_multi.py / clf_1vsall.py
```

## Evaluation signals

- Classification F1-scores after adding photo-z as a feature should reach ~0.992 (galaxies), 0.967 (quasars) and 0.985 (stars) on the 3.5M-source SDSS DR15+WISE benchmark, and must improve relative to running the same classifiers without the photo-z column
- photo_z.py runs without error and every row of the feature table receives a finite photo-z estimate (no rows dropped relative to the create_data.py output)
- The augmented feature table is a valid sparse-tolerant tabular dataset loadable by clf_multi.py / clf_1vsall.py
- Downstream feature_importance.csv ranks the photo-z feature among informative predictors, consistent with the reported performance improvement
- If the input data (including the photo-z feature) changed, FLAML hyper-parameters were re-run rather than reusing the paper's defaults

## Limitations

- The models are optimised for SDSS DR15+WISE; if you use a different dataset you must change the hyper-parameters accordingly and re-run FLAML every time the input data changes
- The chain-regressor provides photometric (not spectroscopic) redshifts, so estimates carry photo-z uncertainty that propagates into classification features
- SHEEP requires tabular input and, while it tolerates sparse data, non-tabular data types are incompatible with the architecture

## Evidence

- [abstract] SHEEP first estimates photometric redshifts, which are then placed into the data set as an additional feature for classification model training; this results in significant improvements in the: "SHEEP first estimates photometric redshifts, which are then placed into the data set as an additional feature for classification model training"
- [readme] The chain-regressor approach used is described in the file "photo_z.py".: "The chain-regressor approach used is described in the file "photo_z.py"."
- [readme] All the feature engineering process can be seen in the file "create_data.py". Make the necessary changes, creating and eliminating features, as needed for your project.: "All the feature engineering process can be seen in the file "create_data.py"."
- [readme] It was built to deal with tabular data, while allowing the use of sparse data.: "It was built to deal with tabular data, while allowing the use of sparse data."
- [methods] The hyper-parameters obtained using FLAML are described. It is recommended to re-run FLAML everytime changes are made in the input data.: "It is recommended to re-run FLAML everytime changes are made in the input data."
