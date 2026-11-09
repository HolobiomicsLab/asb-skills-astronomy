---
name: automated-hyperparameter-optimization
description: Use when training SHEEP's gradient-boosted photo-z regressors or galaxy/QSO/star
  classifiers on SDSS DR15+WISE photometry and tuned hyper-parameters are needed,
  or whenever the input feature table has changed.
when_to_use_negative:
- When reusing the released SHEEP models unchanged on the original SDSS DR15+WISE
  data, where the published FLAML-optimised hyper-parameters already apply
- When the input tabular data differ substantially from SDSS DR15+WISE but you have
  not yet adapted them — optimisation on the wrong feature space will not transfer
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: FLAML
  role: Automated hyper-parameter optimisation of the XGBoost/LightGBM/CatBoost chain-regressor
    and classification models; re-run whenever input data change
  repo: https://github.com/pedro-acunha/SHEEP
- name: XGBoost
  role: Gradient-boosted learner whose hyper-parameters are FLAML-optimised within
    the SHEEP ensemble
- name: LightGBM
  role: Gradient-boosted learner whose hyper-parameters are FLAML-optimised within
    the SHEEP ensemble
- name: CatBoost
  role: Gradient-boosted learner whose hyper-parameters are FLAML-optimised within
    the SHEEP ensemble
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202243135
    title: Photometric redshift-aided classification using ensemble learning
  source_content_hash: sha256:39870b478e16cba33e3c8f0421e8a9892e13fc45499a922b231d054691bbc5bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pedro-acunha/SHEEP
  tools:
  - CatBoost
  - FLAML
  - LightGBM
  - Python
  - SHEEP
  - XGBoost
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# automated-hyperparameter-optimization

## Summary

Optimises the hyper-parameters of the SHEEP pipeline's XGBoost/LightGBM/CatBoost chain-regressor (photo-z) and classifier models using FLAML, and re-runs optimisation whenever input data change.

## When to use

Use when training SHEEP's gradient-boosted photo-z regressors or galaxy/QSO/star classifiers on SDSS DR15+WISE photometry and tuned hyper-parameters are needed, or whenever the input feature table has changed.

## When NOT to use

- When reusing the released SHEEP models unchanged on the original SDSS DR15+WISE data, where the published FLAML-optimised hyper-parameters already apply
- When the input tabular data differ substantially from SDSS DR15+WISE but you have not yet adapted them — optimisation on the wrong feature space will not transfer

## Inputs

- Feature-engineered SDSS DR15+WISE photometry table produced by create_data.py
- Model definitions from photo_z.py, clf_multi.py, or clf_1vsall.py (XGBoost, LightGBM, CatBoost learners)

## Outputs

- FLAML-optimised hyper-parameter configurations for the photo-z chain-regressor and/or classifiers
- Fitted models with tuned hyper-parameters (e.g. the persisted photo-z regressor reused downstream)

## How to apply

Train the chain-regressor ensemble (XGBoost, LightGBM, CatBoost from photo_z.py) and the classifiers (clf_multi.py, clf_1vsall.py), then let FLAML search their hyper-parameters; the resulting tuned values are the ones described in the paper. Because the shipped hyper-parameters were optimised specifically for the SDSS DR15+WISE dataset of 3.5 million sources, re-run FLAML every time changes are made to the input data (e.g. after editing create_data.py feature engineering or swapping in another tabular photometry set). Use the FLAML-optimised hyper-parameters consistently across the photo-z stage and both classification methodologies so the downstream redshift feature and class probabilities are produced by comparably tuned models. Treat this optimisation as a prerequisite step: the fitted regressor persisted for the classification stage must come from FLAML-tuned settings.

## Related tools

- **FLAML** (Automated hyper-parameter optimisation of the XGBoost/LightGBM/CatBoost chain-regressor and classification models; re-run whenever input data change) — https://github.com/pedro-acunha/SHEEP
- **XGBoost** (Gradient-boosted learner whose hyper-parameters are FLAML-optimised within the SHEEP ensemble)
- **LightGBM** (Gradient-boosted learner whose hyper-parameters are FLAML-optimised within the SHEEP ensemble)
- **CatBoost** (Gradient-boosted learner whose hyper-parameters are FLAML-optimised within the SHEEP ensemble)

## Examples

```
python photo_z.py  # fits the XGBoost/LightGBM/CatBoost chain-regressor with FLAML-optimised hyper-parameters; re-run FLAML whenever input data change
```

## Evaluation signals

- The FLAML-derived hyper-parameters differ from the defaults and match the tuned values described in the paper's methods
- The photo-z stage yields complete, finite per-source predictions (photo_z_predictions.csv, one estimate per catalog source) produced by a FLAML-tuned regressor
- Optimisation is re-executed after any modification to the input data, per the README recommendation
- Downstream classification F1-scores remain in the reported range (0.992 galaxies, 0.967 quasars, 0.985 stars) when reproducing on the original SDSS DR15+WISE set

## Limitations

- The shipped hyper-parameters are optimised for SDSS DR15+WISE only; using a different dataset without re-running FLAML can degrade performance
- No explicit numeric photo-z accuracy threshold is reported, so acceptance of the tuned photo-z stage is qualitative (complete, finite per-source predictions)
- Changing feature engineering in create_data.py invalidates prior tuning and requires a fresh FLAML run

## Evidence

- [methods] hyper-parameter optimisation with FLAML: "The hyper-parameters obtained using FLAML are described. It is recommended to re-run FLAML everytime changes are made in the input data."
- [readme] dataset-specific tuning caveat: "If you use it with a different dataset, please remember that the models are optimised for SDSS DR15+WISE. Do not forget to change them according to your dataset."
- [abstract] ensemble learners being tuned: "combines the outputs from the XGBoost, LightGBM, and CatBoost learning algorithms to create stronger classifiers"
- [intro] tuning scope for tabular inputs: "Our approach uses SDSS and WISE photometry, however others types of data (if tabular) are compatible with our architecture, e.g. radio fluxes/magnitudes"
