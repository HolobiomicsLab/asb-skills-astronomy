---
name: multiwavelength-photometric-classification
description: Use when a radio-continuum survey catalog has multi-wavelength photometry
  (optical u–y, NIR JHK, IRAC ch1–ch4, MIPS24, PACS, SPIRE), 150 MHz radio fluxes,
  and redshifts, and each source must be labelled AGN or SFG.
when_to_use_negative:
- Only 1.4 GHz radio data are available and cannot be converted to 150 MHz — the model
  expects 150 MHz radio fluxes and converting 1.4 GHz data reduces performance (~4%
  precision, ~3% recall)
- The task requires calibrated probabilities of AGN membership — the raw output column
  does not correspond to a likelihood that a source is an AGN
- The catalog contains no multi-wavelength photometry or redshifts, e.g. only radio
  morphological information
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_3452
tools:
- name: MBASC
  role: Multi-band AGN-SFG classifier whose LightGBM model is invoked on the input
    catalog to produce AGN/SFG labels
  repo: github.com/Jesper-Karsten/MBASC
- name: LightGBM
  role: Light gradient-boosting machine technique underlying the supervised SFG-vs-AGN
    classification
- name: pandas
  role: Required dependency for loading the CSV catalog and reading output/results.csv
- name: sklearn
  role: Required dependency used for computing per-class precision and recall against
    held-out labels
- name: numpy
  role: Required dependency for the classifier runtime
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202346770
    title: A multi-band AGN-SFG classifier for extragalactic radio surveys using machine
      learning
  source_content_hash: sha256:788bcfad38cb82bb4ebf87f4069d3e3881305365d1cdff169628581f613e9153
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LightGBM
  - MBASC
  - Python
  - numpy
  - pandas
  - sklearn
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# multiwavelength-photometric-classification

## Summary

Supervised classification of radio-continuum sources as AGNs versus star-forming galaxies (SFGs) using a LightGBM-based, multi-band photometric classifier (MBASC) trained on LOFAR Deep Field data with SED-derived labels.

## When to use

Use when a radio-continuum survey catalog has multi-wavelength photometry (optical u–y, NIR JHK, IRAC ch1–ch4, MIPS24, PACS, SPIRE), 150 MHz radio fluxes, and redshifts, and each source must be labelled AGN or SFG.

## When NOT to use

- Only 1.4 GHz radio data are available and cannot be converted to 150 MHz — the model expects 150 MHz radio fluxes and converting 1.4 GHz data reduces performance (~4% precision, ~3% recall)
- The task requires calibrated probabilities of AGN membership — the raw output column does not correspond to a likelihood that a source is an AGN
- The catalog contains no multi-wavelength photometry or redshifts, e.g. only radio morphological information

## Inputs

- CSV catalog with columns named exactly (case-insensitive): id (optional), redshift, u, g, r, i, z, y, J, H, K, ch1–ch4, MIPS24, PACS100, PACS160, SPIRE250, SPIRE350, SPIRE500, radiototal, radiopeak
- All flux densities in μJy (except redshift); 150 MHz radio data (total and peak flux)
- Held-out SED-derived AGN/SFG labels when evaluating precision and recall

## Outputs

- output/results.csv with three columns: id, output (raw LightGBM fraction, 0–1), and AGN (rounded binary: 1 = AGN, 0 = SFG), in the same row order as the input file
- Per-class precision/recall metrics table when evaluated against held-out labels

## How to apply

Assemble a CSV catalog whose headers exactly match MBASC's feature names (redshift, u, g, r, i, z, y, J, H, K, ch1–ch4, MIPS24, PACS100/160, SPIRE250/350/500, radiototal, radiopeak), leaving unavailable bands out rather than renaming them, and convert all fluxes except redshift to μJy. Missing features can simply be omitted, but columns with very few samples should be dropped because sparse data can drive wrong predictions. Run the classifier via `python MBASC.py [path/to/file.csv]`; the raw LightGBM output in output/results.csv is a 0–1 fraction that must be rounded to yield the binary AGN column (1 = AGN, 0 = SFG), and this fraction is not a probability of being an AGN. Validate per-class performance against the published reference metrics (SFG precision 0.92±0.01, recall 0.87±0.02; AGN precision 0.87±0.02, recall 0.78±0.02) computed with sklearn on held-out SED-derived labels, noting that performance degrades at higher redshift due to smaller training samples.

## Related tools

- **MBASC** (Multi-band AGN-SFG classifier whose LightGBM model is invoked on the input catalog to produce AGN/SFG labels) — github.com/Jesper-Karsten/MBASC
- **LightGBM** (Light gradient-boosting machine technique underlying the supervised SFG-vs-AGN classification)
- **pandas** (Required dependency for loading the CSV catalog and reading output/results.csv)
- **sklearn** (Required dependency used for computing per-class precision and recall against held-out labels)
- **numpy** (Required dependency for the classifier runtime)

## Examples

```
python MBASC.py example.csv
```

## Evaluation signals

- Computed per-class precision and recall fall within the reported ranges: SFG precision 0.92±0.01, recall 0.87±0.02; AGN precision 0.87±0.02, recall 0.78±0.02
- Output file output/results.csv exists with exactly three columns (id, output, AGN) and row order matching the input file
- AGN column contains only binary values (1 = AGN, 0 = SFG) derived by rounding the 0–1 raw output
- Input headers match MBASC's exact feature names (capitalisation-insensitive) and all flux units are μJy
- No all-SFG or all-AGN classification pattern, which signals unit-conversion or cross-matching errors

## Limitations

- Raw output column is a fraction, not a likelihood of being an AGN — its interpretation depends on classifier quality over the whole dataset
- Model performance decreases towards higher redshifts, mainly due to smaller training sample sizes
- 1.4 GHz radio data must be manually converted to 150 MHz; using 1.4 GHz directly degrades performance (~4% precision, ~3% recall)
- Wrongly cross-matched radio data and sparse photometric columns can cause misclassifications (e.g. all sources labelled SFG or AGN)
- Far-infrared and radio bands are of great importance — omitting them weakens classification

## Evidence

- [abstract] MBASC trained LightGBM on LOFAR Deep Fields with multi-band photometry: "We performed a supervised classification of SFGs versus AGNs using the light gradient boosting machine (LGBM) on three LOFAR Deep Fields (Lockman Hole, Boötes, and ELAIS-N1)"
- [abstract] Reference precision/recall metrics for SFGs and AGNs: "Our trained model has a precision of 0.92±0.01 and a recall of 0.87±0.02 for SFGs. For AGNs, the model performs slightly worse, with a precision of 0.87±0.02 and a recall of 0.78±0.02."
- [readme] Exact feature names and μJy unit requirement for input CSV: "it is important that the **exact** name (except for capitalisation) is used in the csv file as header. If a certain feature is not available, it can simply be left out. All units (except redshift)"
- [readme] Rounding raw output to binary AGN column; output is not a likelihood: "This value is between 0 and 1 and is simply rounded to result in the AGN column. The AGN column is 1 for a predicted AGN and 0 for a predicted SFG."
- [readme] Troubleshooting: unit conversion and sparse columns cause misclassification: "Check that you converted all the units to μJy. - Remove data columns with very few samples, the model might make wrong predictions based on this."
- [abstract] Performance degrades toward higher redshift: "The model performance decreases towards higher redshifts, which is mainly due to smaller training sample sizes."
- [abstract] 1.4 GHz conversion penalty: "Converting 1.4 GHz to 150 MHz radio data reduces the performance by ~4% in precision and ~3% in recall."
