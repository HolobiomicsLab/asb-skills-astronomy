---
name: lightgbm-classifier-inference
description: Use when a catalog of extragalactic radio-survey sources with multi-band
  photometry (u-y, JHK, IRAC ch1-ch4, MIPS24, PACS, SPIRE, 150 MHz radio) and redshifts
  must be classified as AGN vs SFG and benchmarked.
when_to_use_negative:
- The input is radio data at 1.4 GHz or another waveband and has not been converted
  to 150 MHz — the model was trained on 150 MHz and direct use gives bad classifications
- Photometric fluxes are not in μJy (e.g. still in mJy or Jy) — unit mismatch is the
  README's primary cause of systematic misclassification
- The raw 'output' fraction is needed as a probability or likelihood of being an AGN
  — the README explicitly warns it does not correspond to a likelihood and depends
  on classifier performance on the whole dataset
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3360
- http://edamontology.org/topic_3474
tools:
- name: MBASC
  role: The classifier script (MBASC.py) that loads the feature CSV, runs the trained
    LightGBM model, and writes output/results.csv with binary AGN/SFG labels
  repo: https://github.com/Jesper-Karsten/MBASC
- name: LightGBM
  role: Light gradient boosting machine algorithm underlying MBASC; produces the raw
    0-1 output fraction that is rounded to the binary AGN column
- name: pandas
  role: Load and validate the held-out/input catalog, checking exact header names
    and μJy flux units
- name: sklearn
  role: Compute per-class precision and recall for SFG and AGN classes on held-out
    labels
- name: numpy
  role: Dependency required to run the MBASC algorithm
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202346770
    title: A multi-band AGN-SFG classifier for extragalactic radio surveys using machine
      learning
  source_content_hash: sha256:fb9d1de93496b6b9912330f977332ebc7b719610ae924dc77ca7f6f2e64c830c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Jesper-Karsten/MBASC
  tools:
  - LightGBM
  - MBASC
  - Python
  - numpy
  - pandas
  - sklearn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3360
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# lightgbm-classifier-inference

## Summary

Run the MBASC LightGBM classifier to label radio-survey sources as AGNs (1) or SFGs (0) from multi-wavelength fluxes and redshifts, and compute per-class precision/recall against SED-derived held-out labels.

## When to use

Use when a catalog of extragalactic radio-survey sources with multi-band photometry (u-y, JHK, IRAC ch1-ch4, MIPS24, PACS, SPIRE, 150 MHz radio) and redshifts must be classified as AGN vs SFG and benchmarked.

## When NOT to use

- The input is radio data at 1.4 GHz or another waveband and has not been converted to 150 MHz — the model was trained on 150 MHz and direct use gives bad classifications
- Photometric fluxes are not in μJy (e.g. still in mJy or Jy) — unit mismatch is the README's primary cause of systematic misclassification
- The raw 'output' fraction is needed as a probability or likelihood of being an AGN — the README explicitly warns it does not correspond to a likelihood and depends on classifier performance on the whole dataset

## Inputs

- CSV file with columns matching MBASC feature names: redshift, u, g, r, i, z, y, J, H, K, ch1-ch4 (IRAC 3.6-7.9 μm), MIPS24, PACS100, PACS160, SPIRE250, SPIRE350, SPIRE500, radiototal, radiopeak (all fluxes in μJy)
- Held-out catalog with SED-derived AGN/SFG labels for metric computation (required for the validation variant)
- Optional 'id' column to preserve source identifiers in the output

## Outputs

- output/results.csv with three columns: id, output (raw LightGBM fraction 0-1), and AGN (1 = AGN, 0 = SFG), in the same order as the input file
- Per-class precision and recall table for SFG and AGN classes (when held-out labels are supplied)

## How to apply

Prepare a CSV whose headers exactly match MBASC feature names (redshift, u, g, r, i, z, y, J, H, K, ch1-ch4, MIPS24, PACS100/160, SPIRE250/350/500, radiototal, radiopeak), converting all fluxes except redshift to μJy; unavailable features can simply be left out. Run the classifier with 'python MBASC.py [path/to/file.csv]', which writes output/results.csv with id, output (raw LightGBM fraction between 0 and 1), and AGN columns — the AGN column is the raw output rounded to a binary label (1 = AGN, 0 = SFG). Note the raw output is not a likelihood the source is an AGN. To reproduce reported performance, compute per-class precision and recall with sklearn on held-out SED-derived labels and compare against the reference values (SFG precision 0.92±0.01, recall 0.87±0.02; AGN precision 0.87±0.02, recall 0.78±0.02). Use 150 MHz radio data (radiototal, radiopeak); 1.4 GHz data can be converted, but this degrades performance by ~4% in precision and ~3% in recall.

## Related tools

- **MBASC** (The classifier script (MBASC.py) that loads the feature CSV, runs the trained LightGBM model, and writes output/results.csv with binary AGN/SFG labels) — https://github.com/Jesper-Karsten/MBASC
- **LightGBM** (Light gradient boosting machine algorithm underlying MBASC; produces the raw 0-1 output fraction that is rounded to the binary AGN column)
- **pandas** (Load and validate the held-out/input catalog, checking exact header names and μJy flux units)
- **sklearn** (Compute per-class precision and recall for SFG and AGN classes on held-out labels)
- **numpy** (Dependency required to run the MBASC algorithm)

## Examples

```
python MBASC.py path/to/catalog.csv && python -c "import pandas as pd, sklearn.metrics as m; df=pd.read_csv('output/results.csv'); print(m.precision_score(df.AGN, labels), m.recall_score(df.AGN, labels))"
```

## Evaluation signals

- Computed SFG precision and recall fall within the reported ranges 0.92±0.01 and 0.87±0.02
- Computed AGN precision and recall fall within the reported ranges 0.87±0.02 and 0.78±0.02
- output/results.csv contains exactly the columns id, output, and AGN, with rows in the same order as the input file
- AGN column values are binary (0 or 1) and equal the raw output fraction rounded; raw output values lie between 0 and 1
- Input CSV headers match the required feature names exactly (case-insensitive) and all fluxes are in μJy

## Limitations

- The raw output column does not have to correspond to a likelihood that a source is an AGN; it depends on classifier performance on the whole dataset
- Converting 1.4 GHz to 150 MHz radio data reduces performance by ~4% in precision and ~3% in recall
- Model performance decreases towards higher redshifts, mainly due to smaller training sample sizes
- Data columns with very few samples, wrongly cross-matched radio data, or non-μJy units can cause systematic misclassification (e.g. all sources labelled SFG or AGN)

## Evidence

- [methods] MBASC invocation and rounding rule: [evidence span withheld — anchor status: not_found]
- [readme] Feature and unit requirements: "it is important that the **exact** name (except for capitalisation) is used in the csv file as header. If a certain feature is not available, it can simply be left out. All units (except redshift)"
- [readme] Output file schema: "The results.csv file has 3 columns: id, output, and AGN. The ID column is the given ID, or if not given a simple numeric value. The results.csv is in the same order as the input file."
- [abstract] Reference precision/recall values: "Our trained model has a precision of 0.92±0.01 and a recall of 0.87±0.02 for SFGs. For AGNs, the model performs slightly worse, with a precision of 0.87±0.02 and a recall of 0.78±0.02."
- [readme] Output is not a likelihood: "the output column **does not** have to correspond to a likelihood that a source is an AGN. It entirely depends on how well the classifier works on the whole dataset."
