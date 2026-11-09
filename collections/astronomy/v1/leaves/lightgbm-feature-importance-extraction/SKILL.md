---
name: lightgbm-feature-importance-extraction
description: Use when a trained MBASC/LightGBM classifier exists on multi-wavelength
  flux data and you need to rank which photometric bands (e.g. FIR and radio) most
  influence AGN vs SFG predictions.
when_to_use_negative:
- The input catalogue is not in MBASC's expected schema or fluxes are not in μJy —
  the booster would attach importances to mis-specified features
- You need calibrated likelihoods rather than relative feature rankings — the README
  warns the raw output column does not have to correspond to a likelihood that a source
  is an AGN
- The goal is AGN/SFG classification itself rather than model interpretation — run
  MBASC directly without the importance-extraction step
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: lightgbm
  role: Provides the gradient-boosting model whose fitted booster exposes the per-feature
    importance API queried in this skill
  repo: https://github.com/Jesper-Karsten/MBASC
- name: MBASC
  role: Trains the LGBM AGN/SFG classifier whose booster is analysed for feature importances
  repo: https://github.com/Jesper-Karsten/MBASC
- name: pandas
  role: Loads the μJy flux catalogue CSV and assembles the ranked feature-importance
    table for export
- name: numpy
  role: Handles the numeric importance-score arrays returned by the LightGBM feature-importance
    API
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202346770
    title: A multi-band AGN-SFG classifier for extragalactic radio surveys using machine
      learning
  source_content_hash: sha256:dc3ce589e6deb0a42aaea4bd5627a8f42bb66096e2807a887855b7cbb0b805f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Jesper-Karsten/MBASC
  tools:
  - MBASC
  - Python
  - lightgbm
  - numpy
  - pandas
  - sklearn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# lightgbm-feature-importance-extraction

## Summary

Extract and rank per-feature importance scores from the fitted LightGBM booster of the MBASC AGN/SFG classifier, quantifying which of the 23 multi-wavelength photometric bands (u through SPIRE500, radiototal, radiopeak) and redshift drive the classification.

## When to use

Use when a trained MBASC/LightGBM classifier exists on multi-wavelength flux data and you need to rank which photometric bands (e.g. FIR and radio) most influence AGN vs SFG predictions.

## When NOT to use

- The input catalogue is not in MBASC's expected schema or fluxes are not in μJy — the booster would attach importances to mis-specified features
- You need calibrated likelihoods rather than relative feature rankings — the README warns the raw output column does not have to correspond to a likelihood that a source is an AGN
- The goal is AGN/SFG classification itself rather than model interpretation — run MBASC directly without the importance-extraction step

## Inputs

- Multi-wavelength flux catalogue CSV (e.g. example.csv) with flux densities in μJy and headers u, g, r, i, z, y, J, H, K, ch1-ch4, MIPS24, PACS100, PACS160, SPIRE250, SPIRE350, SPIRE500, radiototal, radiopeak, redshift
- Fitted LightGBM booster from the MBASC classifier (trained via python MBASC.py)

## Outputs

- Ranked feature-importance table (e.g. feature_importances.csv) ordering the 23 features by importance score
- Raw model predictions in output/results.csv (columns: id, output, AGN) from the MBASC run that produced the booster

## How to apply

Load the multi-wavelength flux catalogue (e.g. example.csv from the MBASC repository) with pandas, ensuring all flux densities are converted to μJy and headers use the exact feature names (u, g, r, i, z, y, J, H, K, ch1–ch4, MIPS24, PACS100, PACS160, SPIRE250, SPIRE350, SPIRE500, radiototal, radiopeak, redshift). Train the LightGBM classifier by running MBASC (python MBASC.py example.csv), which writes raw predictions to output/results.csv. Then query the fitted LightGBM booster's feature-importance API via numpy arrays to obtain per-feature scores. Rank all 23 features by importance score and export the ranked table (e.g. feature_importances.csv). Validate by confirming that the far-infrared bands (MIPS24, PACS100, PACS160, SPIRE250/350/500) and radio bands (radiototal, radiopeak) appear among the top-ranked features, matching the paper's finding.

## Related tools

- **lightgbm** (Provides the gradient-boosting model whose fitted booster exposes the per-feature importance API queried in this skill) — https://github.com/Jesper-Karsten/MBASC
- **MBASC** (Trains the LGBM AGN/SFG classifier whose booster is analysed for feature importances) — https://github.com/Jesper-Karsten/MBASC
- **pandas** (Loads the μJy flux catalogue CSV and assembles the ranked feature-importance table for export)
- **numpy** (Handles the numeric importance-score arrays returned by the LightGBM feature-importance API)

## Examples

```
python MBASC.py example.csv; then read output/results.csv and extract booster.feature_importance() with numpy to build feature_importances.csv
```

## Evaluation signals

- Far-infrared bands (MIPS24, PACS100, PACS160, SPIRE250, SPIRE350, SPIRE500) and radio bands (radiototal, radiopeak) appear among the top-ranked features
- The ranked table contains all 23 expected features with no missing or duplicated names
- Importance scores are non-negative and rank-ordered as returned by the LightGBM booster's feature-importance API
- The MBASC run that produced the booster completed and wrote output/results.csv in the same row order as the input file
- All fluxes were verified to be in μJy before training, since wrong units can distort which features the model relies on

## Limitations

- MBASC expects 150 MHz radio data (radiototal, radiopeak); using 1.4 GHz or other wavebands without conversion can give wrong classifications and importances
- The README warns that the raw LightGBM output value does not have to correspond to a likelihood that a source is an AGN — importance extraction must not be confused with probability calibration
- Data columns with very few samples, or wrongly cross-matched radio data, can make the model rely on spurious features, distorting the importance ranking

## Evidence

- [abstract] we find that the far-infrared and radio bands are of great importance.: "we find that the far-infrared and radio bands are of great importance."
- [readme] An example file is included in this repository (example.csv) on which you can run the algorithm (python MBASC.py example.csv).: "An example file is included in this repository (example.csv) on which you can run the algorithm (python MBASC.py example.csv)."
- [readme] All units (except redshift) must be converted to μJy.: "All units (except redshift) must be converted to μJy."
- [methods] the following dependecies must be installed: - pandas - lightgbm - sklearn - numpy: "the following dependecies must be installed: - pandas - lightgbm - sklearn - numpy"
