---
name: lgbm-gradient-boosting-classifier-training
description: Use when a labeled multi-wavelength photometric catalog (fluxes in μJy
  plus redshift) with AGN/SFG classes exists and a fast, reproducible binary classifier
  for radio continuum survey sources is needed.
when_to_use_negative:
- Inputs are spectroscopic classifications or full SED fits rather than multi-band
  photometric fluxes with redshift — MBASC is designed for photometric catalogs only.
- Data columns contain very few samples or fluxes are not in μJy — the README warns
  the model may make wrong predictions based on sparse columns or unit errors.
- Radio data is 1.4 GHz (unconverted) or wrongly cross-matched; the README states
  wrong cross-matches and non-150 MHz radio data give bad results.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: LightGBM
  role: Gradient-boosted ML algorithm used for the supervised AGN-vs-SFG classification
  repo: github.com/Jesper-Karsten/MBASC
- name: pandas
  role: Loads the photometric CSV catalog with exact feature headers
- name: lightgbm
  role: Python dependency that provides the LGBM classifier implementation
- name: sklearn
  role: Train/test splitting and per-class precision/recall computation
- name: numpy
  role: Numerical dependency for feature array handling
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202346770
    title: A multi-band AGN-SFG classifier for extragalactic radio surveys using machine
      learning
  source_content_hash: sha256:c7f9f7c9fa01a9aff7daf0ed8a6e2aadc04a9be2b1fa5de32c1796d26024153e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LightGBM
  - MBASC
  - Python
  - lightgbm
  - numpy
  - pandas
  - sklearn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# lgbm-gradient-boosting-classifier-training

## Summary

Trains a supervised LightGBM (LGBM) classifier to distinguish AGNs from star-forming galaxies (SFGs) using multi-wavelength photometry (u through SPIRE500, IRAC ch1–ch4, MIPS24, PACS, 150 MHz radio) plus redshift, with SED-derived labels from the three LOFAR Deep Fields.

## When to use

Use when a labeled multi-wavelength photometric catalog (fluxes in μJy plus redshift) with AGN/SFG classes exists and a fast, reproducible binary classifier for radio continuum survey sources is needed.

## When NOT to use

- Inputs are spectroscopic classifications or full SED fits rather than multi-band photometric fluxes with redshift — MBASC is designed for photometric catalogs only.
- Data columns contain very few samples or fluxes are not in μJy — the README warns the model may make wrong predictions based on sparse columns or unit errors.
- Radio data is 1.4 GHz (unconverted) or wrongly cross-matched; the README states wrong cross-matches and non-150 MHz radio data give bad results.

## Inputs

- CSV photometric catalog with exact headers: redshift, u, g, r, i, z, y, J, H, K, ch1–ch4 (IRAC 3.6–7.9 μm), MIPS24, PACS100/160, SPIRE250/350/500, radiototal, radiopeak; all fluxes in μJy
- SED-derived AGN/SFG training labels from the three LOFAR Deep Fields (Lockman Hole, Boötes, ELAIS-N1)
- 150 MHz LOFAR radio continuum fluxes (or 1.4 GHz data converted to 150 MHz by the user)

## Outputs

- Fitted LightGBM model file (lgbm_agn_sfg_model.txt)
- Raw 0–1 LightGBM output column and rounded binary AGN column per source (output/results.csv with id, output, AGN)
- Per-class precision and recall metrics table (SFG: 0.92±0.01 / 0.87±0.02; AGN: 0.87±0.02 / 0.78±0.02)

## How to apply

Load the LOFAR Deep Fields photometric catalog (Lockman Hole, Boötes, ELAIS-N1) with pandas, keeping exact feature headers (redshift, u/g/r/i/z/y, J/H/K, ch1–ch4, MIPS24, PACS100/160, SPIRE250/350/500, radiototal/radiopeak) and converting every flux to μJy. Assemble the SED-derived AGN/SFG labels and split into training and test sets with scikit-learn, then fit the lightgbm classifier on the multi-band features plus redshift. Predict raw 0–1 outputs on the held-out test set and round them to binary AGN (1) / SFG (0) classes — note the raw output is not a probability that a source is an AGN. Report per-class precision and recall with sklearn; the trained model achieved precision 0.92±0.01 / recall 0.87±0.02 for SFGs and 0.87±0.02 / 0.78±0.02 for AGNs, with far-infrared and radio bands being of great importance to performance.

## Related tools

- **LightGBM** (Gradient-boosted ML algorithm used for the supervised AGN-vs-SFG classification) — github.com/Jesper-Karsten/MBASC
- **pandas** (Loads the photometric CSV catalog with exact feature headers)
- **lightgbm** (Python dependency that provides the LGBM classifier implementation)
- **sklearn** (Train/test splitting and per-class precision/recall computation)
- **numpy** (Numerical dependency for feature array handling)

## Examples

```
python MBASC.py example.csv
```

## Evaluation signals

- Per-class metrics reach the published range: SFG precision 0.92±0.01 / recall 0.87±0.02; AGN precision 0.87±0.02 / recall 0.78±0.02 on the held-out test set.
- Raw LightGBM output values lie between 0 and 1 and round to a binary AGN column (1 = AGN, 0 = SFG) in output/results.csv, in the same row order as the input file.
- Model performance decreases towards higher redshifts, mainly due to smaller training sample sizes — a z-stratified check should reproduce this trend.
- Feature-importance inspection shows far-infrared and radio bands rank as the most important bands for classification.
- All flux units verified as μJy (unit errors are the README's first troubleshooting item for misclassifications).

## Limitations

- The raw output column does not have to correspond to a likelihood that a source is an AGN; it depends on how well the classifier works on the whole dataset.
- Model performance decreases towards higher redshifts, mainly due to smaller training sample sizes.
- Converting 1.4 GHz to 150 MHz radio data reduces performance by ~4% in precision and ~3% in recall.
- Wrongly cross-matched radio data or sparse data columns can lead to systematically wrong (or all-SFG/all-AGN) classifications.
- The model expects 150 MHz radio data specifically; 1.4 GHz input must be converted by the user (MBASC does not do this itself).

## Evidence

- [abstract] Supervised LGBM training on LOFAR Deep Fields: "We performed a supervised classification of SFGs versus AGNs using the light gradient boosting machine (LGBM) on three LOFAR Deep Fields (Lockman Hole, Boötes, and ELAIS-N1)"
- [abstract] Published precision/recall metrics: "Our trained model has a precision of 0.92±0.01 and a recall of 0.87±0.02 for SFGs. For AGNs, the model performs slightly worse, with a precision of 0.87±0.02 and a recall of 0.78±0.02."
- [abstract] Informative bands: "we find that the far-infrared and radio bands are of great importance."
- [readme] Raw output interpretation caveat: "the output column **does not** have to correspond to a likelihood that a source is an AGN"
- [readme] Unit requirement and feature headers: "it is important that the **exact** name (except for capitalisation) is used in the csv file as header. If a certain feature is not available, it can simply be left out. All units (except redshift)"
- [methods] Binary rounding of the output: "This value is between 0 and 1 and is simply rounded to result in the AGN column."
