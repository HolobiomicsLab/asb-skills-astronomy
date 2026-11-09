---
name: photo-z-estimate-feature-augmentation
description: Use when classifying astronomical sources from tabular photometry (e.g.
  SDSS DR15+WISE magnitudes/fluxes of millions of sources) and you want to boost classifier
  accuracy by adding photometric redshifts as an engineered feature before training.
when_to_use_negative:
- The downstream classifiers expect the exact input data SHEEP was optimised for (SDSS
  DR15+WISE); with a different dataset you must re-run FLAML and retune rather than
  assuming the photo-z-augmented setup transfers unchanged
- Data are not tabular (e.g. images or spectra) — SHEEP's architecture is built for
  tabular data, though other tabular inputs such as radio fluxes/magnitudes are compatible
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_2640
tools:
- name: SHEEP
  role: Pipeline containing the chain-regressor photo-z estimation step (photo_z.py)
    whose output is added as a classification feature
  repo: github.com/pedro-acunha/SHEEP
- name: Python
  role: Language in which photo_z.py and the rest of the pipeline are implemented
- name: Jupyter Notebook
  role: Recommended environment for running and visualising the photo-z and classification
    workflow
- name: FLAML
  role: Re-run hyper-parameter optimisation after photo-z augmentation so the classifiers
    are re-tuned for the changed input data
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202243135
    title: Photometric redshift-aided classification using ensemble learning
  source_content_hash: sha256:29fb5e2c59ee7b5c4edd8b19930169a1b971bb92493e85b6baee3be94276fb5f
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
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Photo-z estimate feature augmentation

## Summary

Estimate photometric redshifts from SDSS+WISE tabular photometry with a chain-regressor (photo_z.py) and append the predicted redshifts to the feature table as an additional input to the galaxy/quasar/star classification models, which yields significant improvements in classification performance.

## When to use

Use when classifying astronomical sources from tabular photometry (e.g. SDSS DR15+WISE magnitudes/fluxes of millions of sources) and you want to boost classifier accuracy by adding photometric redshifts as an engineered feature before training.

## When NOT to use

- The downstream classifiers expect the exact input data SHEEP was optimised for (SDSS DR15+WISE); with a different dataset you must re-run FLAML and retune rather than assuming the photo-z-augmented setup transfers unchanged
- Data are not tabular (e.g. images or spectra) — SHEEP's architecture is built for tabular data, though other tabular inputs such as radio fluxes/magnitudes are compatible

## Inputs

- Tabular photometry feature table from SDSS DR15+WISE (created via sql_query_casjobs and create_data.py; sparse data allowed)
- Trained/configured chain-regressor photo-z model (photo_z.py)

## Outputs

- Photo-z estimates for each astronomical source
- Augmented feature table with photometric redshift appended as an additional classification feature

## How to apply

Before running the classification, apply the chain-regressor approach implemented in SHEEP's 'photo_z.py' to the engineered photometry table produced by 'create_data.py' (SDSS DR15+WISE magnitudes/fluxes of ~3.5 million sources, sparse values allowed). Append the estimated photo-z values as an extra column in the dataset so they are available to the downstream classifiers (XGBoost, LightGBM, CatBoost combined in clf_multi.py / clf_1vsall.py). Because the models are optimised for SDSS DR15+WISE, re-run the FLAML hyper-parameter search after augmenting the data with photo-z so the hyper-parameters adapt to the new feature. The rationale is that redshift information strongly separates stars (no redshift) from extragalactic sources (galaxies vs quasars), leading to significant gains in per-class F1-scores.

## Related tools

- **SHEEP** (Pipeline containing the chain-regressor photo-z estimation step (photo_z.py) whose output is added as a classification feature) — github.com/pedro-acunha/SHEEP
- **Python** (Language in which photo_z.py and the rest of the pipeline are implemented)
- **Jupyter Notebook** (Recommended environment for running and visualising the photo-z and classification workflow)
- **FLAML** (Re-run hyper-parameter optimisation after photo-z augmentation so the classifiers are re-tuned for the changed input data)

## Examples

```
python photo_z.py  # estimate photo-z, then append values to the dataset before running clf_multi.py
```

## Evaluation signals

- Photo-z values are successfully appended as an additional feature column in the dataset prior to classification model training
- Downstream classification F1-scores improve relative to training without the photo-z feature (the paper reports significant improvements from photo-z augmentation)
- Final per-class F1-scores on the 3.5M-source SDSS+WISE set reach ~0.992 (galaxies), 0.967 (quasars), 0.985 (stars)
- FLAML hyper-parameter search was re-run after the input data changed, as recommended by the README

## Limitations

- Hyper-parameters obtained with FLAML are optimised for SDSS DR15+WISE; using a different dataset requires re-running the optimisation
- The photo-z gains are demonstrated on SDSS+WISE photometry; transfer to other tabular data types (e.g. radio fluxes/magnitudes) is architecturally supported but not benchmarked here
- Sparse photometry is allowed, but sparse/mising bands may degrade photo-z estimates (inherent to photometric redshift estimation)

## Evidence

- [abstract] prior to performing the classification, SHEEP first estimates photometric redshifts, which are then placed into the data set as an additional feature: "prior to performing the classification, SHEEP first estimates photometric redshifts, which are then placed into the data set as an additional feature"
- [abstract] Estimating photometric redshifts and adding them as a classification feature results in significant improvements in classification performance: [evidence span withheld — anchor status: not_found]
- [readme] The chain-regressor approach used is described in the file "photo_z.py".: "The chain-regressor approach used is described in the file "photo_z.py"."
- [readme] It is recommended to re-run FLAML everytime changes are made in the input data.: "It is recommended to re-run FLAML everytime changes are made in the input data."
