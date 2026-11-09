---
name: multi-wavelength-sed-feature-preparation
description: 'Use when building the input CSV for MBASC or a similar LightGBM AGN/SFG
  classifier: you have multi-band photometry (optical/NIR/MIR/FIR/radio) with redshift
  that must be assembled into the exact expected feature schema in μJy.'
when_to_use_negative:
- You only need a classifier prediction and already have a schema-conformant μJy feature
  table — run MBASC directly instead of re-preparing features
- Your data are single-band or lack multi-wavelength coverage and redshift — MBASC
  expects a wide range of multi-wavelength data and redshifts to predict a classification
- You want the raw LightGBM output treated as a probability — the output column does
  not have to correspond to a likelihood that a source is an AGN
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3360
tools:
- name: pandas
  role: Load and assemble the photometric catalog CSV with exact feature headers before
    running the classifier
  repo: https://github.com/Jesper-Karsten/MBASC
- name: numpy
  role: Numerical handling of flux arrays and unit conversion during feature preparation
  repo: https://github.com/Jesper-Karsten/MBASC
- name: lightgbm
  role: Consumes the prepared multi-band features plus redshift as classifier input
  repo: https://github.com/Jesper-Karsten/MBASC
- name: MBASC
  role: The multi-band AGN-SFG classifier whose input schema dictates the exact feature
    names and μJy units
  repo: https://github.com/Jesper-Karsten/MBASC
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202346770
    title: A multi-band AGN-SFG classifier for extragalactic radio surveys using machine
      learning
  source_content_hash: sha256:adfd53f6c37abf8d76494ba96a7c52ed9c3019b27f18769e29d6b78e2f50811f
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# multi-wavelength-sed-feature-preparation

## Summary

Prepares multi-wavelength photometric feature tables (u through K, IRAC ch1–ch4, MIPS24, PACS100/160, SPIRE250/350/500, radiototal/radiopeak, plus redshift) for the MBASC LightGBM AGN/SFG classifier, ensuring exact header names and μJy flux units.

## When to use

Use when building the input CSV for MBASC or a similar LightGBM AGN/SFG classifier: you have multi-band photometry (optical/NIR/MIR/FIR/radio) with redshift that must be assembled into the exact expected feature schema in μJy.

## When NOT to use

- You only need a classifier prediction and already have a schema-conformant μJy feature table — run MBASC directly instead of re-preparing features
- Your data are single-band or lack multi-wavelength coverage and redshift — MBASC expects a wide range of multi-wavelength data and redshifts to predict a classification
- You want the raw LightGBM output treated as a probability — the output column does not have to correspond to a likelihood that a source is an AGN

## Inputs

- Photometric catalog (CSV) with optical (u,g,r,i,z,y), NIR (J,H,K), IRAC ch1–ch4, MIPS 24 μm, PACS 100/160 μm, SPIRE 250/350/500 μm flux densities in μJy
- 150 MHz radio continuum fluxes (radiototal, radiopeak) in μJy, or 1.4 GHz fluxes converted to 150 MHz
- Redshift per source (unitless, not converted)
- Optional per-source id column

## Outputs

- Feature CSV with exact MBASC header names (e.g. example.csv format) ready for `python MBASC.py <file.csv>`
- Downstream output/results.csv with id, raw LightGBM output (0–1), and binary AGN column

## How to apply

Assemble a CSV whose header uses the exact feature names required by MBASC — redshift, u/g/r/i/z/y, J/H/K, IRAC ch1–ch4, MIPS24, PACS100, PACS160, SPIRE250, SPIRE350, SPIRE500, radiototal, radiopeak — matching names except for capitalisation; optional features can simply be omitted, and an optional id column controls output ordering. Convert every flux (except redshift) to μJy, since wrong units are a documented failure mode that produces all-SFG or all-AGN classifications. Supply 150 MHz radio fluxes (radiototal, radiopeak); if only 1.4 GHz data are available they must be converted to 150 MHz yourself, at a cost of ~4% precision and ~3% recall. Drop columns with very few samples and be wary of badly cross-matched radio data, which can drive wrong predictions. Validate the table against example.csv and by running `python MBASC.py example.csv` before applying to real catalogs.

## Related tools

- **pandas** (Load and assemble the photometric catalog CSV with exact feature headers before running the classifier) — https://github.com/Jesper-Karsten/MBASC
- **numpy** (Numerical handling of flux arrays and unit conversion during feature preparation) — https://github.com/Jesper-Karsten/MBASC
- **lightgbm** (Consumes the prepared multi-band features plus redshift as classifier input) — https://github.com/Jesper-Karsten/MBASC
- **MBASC** (The multi-band AGN-SFG classifier whose input schema dictates the exact feature names and μJy units) — https://github.com/Jesper-Karsten/MBASC

## Examples

```
python MBASC.py example.csv
```

## Evaluation signals

- Header names match the MBASC feature table exactly (except capitalisation); missing features are omitted rather than renamed
- All flux columns are in μJy (redshift excluded); wrong units are a documented cause of degenerate all-SFG/all-AGN predictions
- Radio columns contain 150 MHz fluxes, not raw 1.4 GHz values; converted 1.4 GHz data should cost ~4% precision and ~3% recall
- Sanity-check run `python MBASC.py example.csv` completes and produces output/results.csv with id, output, and AGN columns in input order
- Columns with very few samples are removed, and no obviously mis-cross-matched radio fluxes remain

## Limitations

- Raw output values are fractions between 0 and 1 but are not likelihoods that a source is an AGN; their interpretation depends on how well the classifier works on the whole dataset
- Using 1.4 GHz radio data (converted to 150 MHz) reduces performance by ~4% in precision and ~3% in recall
- Removing or mis-converting units leads to systematic misclassification; columns with few samples and wrongly cross-matched radio data can produce wrong predictions
- Trained on the three LOFAR Deep Fields (Lockman Hole, Boötes, ELAIS-N1); performance decreases towards higher redshifts due to smaller training sample sizes

## Evidence

- [readme] exact feature names and μJy units required: "it is important that the **exact** name (except for capitalisation) is used in the csv file as header. If a certain feature is not available, it can simply be left out. All units (except redshift)"
- [readme] feature table scope: "This model can use a wide range of multi-wavelength data and redshifts to predict a classification for sources."
- [readme] unit-conversion failure mode: "Check that you converted all the units to μJy. - Remove data columns with very few samples, the model might make wrong predictions based on this."
- [abstract] 1.4 GHz conversion caveat: "Converting 1.4 GHz to 150 MHz radio data reduces the performance by ~4% in precision and ~3% in recall."
- [abstract] training data and labels: "We performed a supervised classification of SFGs versus AGNs using the light gradient boosting machine (LGBM) on three LOFAR Deep Fields (Lockman Hole, Boötes, and ELAIS-N1)"
