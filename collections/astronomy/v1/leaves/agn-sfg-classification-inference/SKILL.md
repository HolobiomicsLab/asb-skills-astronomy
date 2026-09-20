---
name: agn-sfg-classification-inference
description: Use when you have a catalog CSV of extragalactic radio-survey sources
  with multi-band photometric flux densities (in μJy) and redshifts, and need AGN/SFG
  classifications via a trained LightGBM model.
when_to_use_negative:
- The input fluxes are at 1.4 GHz or another waveband and have not been converted
  to 150 MHz — the model will misclassify until converted
- Columns contain very few samples or wrongly cross-matched radio data — the model
  may make wrong predictions based on these; remove or clean them first instead of
  trusting inference results
- You need calibrated probabilities of AGN likelihood — the raw output column explicitly
  does not correspond to a likelihood that a source is an AGN
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: MBASC
  role: The trained LightGBM-based classifier run as 'python MBASC.py <file.csv>'
    to produce AGN/SFG predictions
  repo: https://github.com/Jesper-Karsten/MBASC
- name: lightgbm
  role: Underlying light gradient-boosting machine technique and required dependency
    providing the model inference
- name: pandas
  role: Required dependency for loading and validating the input CSV catalog
- name: numpy
  role: Required dependency supporting the classification pipeline
- name: sklearn
  role: Required dependency of the MBASC algorithm
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1051/0004-6361/202346770
    title: A multi-band AGN-SFG classifier for extragalactic radio surveys using machine
      learning
  source_content_hash: sha256:20fa7b13497874a6c4ac03c31f95e3014117c21556ec9853b9483b696d833726
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# agn-sfg-classification-inference

## Summary

Run the MBASC LightGBM-based classifier to predict binary AGN vs SFG labels for radio-survey sources from a CSV of multi-wavelength flux densities (u through SPIRE500, plus radio and redshift), and interpret the raw fractional output correctly.

## When to use

Use when you have a catalog CSV of extragalactic radio-survey sources with multi-band photometric flux densities (in μJy) and redshifts, and need AGN/SFG classifications via a trained LightGBM model.

## When NOT to use

- The input fluxes are at 1.4 GHz or another waveband and have not been converted to 150 MHz — the model will misclassify until converted
- Columns contain very few samples or wrongly cross-matched radio data — the model may make wrong predictions based on these; remove or clean them first instead of trusting inference results
- You need calibrated probabilities of AGN likelihood — the raw output column explicitly does not correspond to a likelihood that a source is an AGN

## Inputs

- CSV file with multi-wavelength flux density columns named exactly as MBASC features (u, g, r, i, z, y, J, H, K, ch1, ch2, ch3, ch4, MIPS24, PACS100, PACS160, SPIRE250, SPIRE350, SPIRE500, radiototal, radiopeak) plus optional id and redshift
- All flux densities converted to μJy; radio fluxes at 150 MHz (radiototal, radiopeak)
- example.csv from the Jesper-Karsten/MBASC repository as a template/reference

## Outputs

- output/results.csv with three columns: id, output (raw LightGBM fraction between 0 and 1), and AGN (binary, rounded: 1 = AGN, 0 = SFG), in the same order as the input file

## How to apply

Load the input catalog with pandas ensuring flux densities are converted to μJy (all units except redshift) and that column headers exactly match MBASC feature names (u, g, r, i, z, y, J, H, K, ch1-ch4, MIPS24, PACS100/160, SPIRE250/350/500, radiototal, radiopeak); unavailable features can simply be left out. Run the classifier with 'python MBASC.py [path/to/file.csv]', which reads the LightGBM model trained on three LOFAR Deep Fields (Lockman Hole, Boötes, ELAIS-N1) and writes output/results.csv containing id, output, and AGN columns in the same order as the input. Round the raw fractional LightGBM output (0-1) to produce the binary AGN column (1 = AGN, 0 = SFG). Critically, do not interpret the raw output column as a likelihood that a source is an AGN — it depends on how well the classifier works on the whole dataset. If the radio data is 1.4 GHz rather than 150 MHz, convert it yourself before running, since the model expects 150 MHz radio flux.

## Related tools

- **MBASC** (The trained LightGBM-based classifier run as 'python MBASC.py <file.csv>' to produce AGN/SFG predictions) — https://github.com/Jesper-Karsten/MBASC
- **lightgbm** (Underlying light gradient-boosting machine technique and required dependency providing the model inference)
- **pandas** (Required dependency for loading and validating the input CSV catalog)
- **numpy** (Required dependency supporting the classification pipeline)
- **sklearn** (Required dependency of the MBASC algorithm)

## Examples

```
python MBASC.py example.csv
```

## Evaluation signals

- output/results.csv exists and has exactly the three columns id, output, and AGN
- output values are fractions between 0 and 1; AGN values are binary (1 = AGN, 0 = SFG) and equal the rounded output
- Rows in results.csv are in the same order as the input file
- Running on the bundled example.csv (python MBASC.py example.csv) completes without errors, confirming column names and μJy units are correct
- On reference data the classifier should perform near the reported precision/recall (SFG precision 0.92, recall 0.87; AGN precision 0.87, recall 0.78); systematically all-SFG or all-AGN output indicates unit or waveband errors

## Limitations

- The raw output column does not have to correspond to a likelihood that a source is an AGN; it depends on classifier performance on the whole dataset, so use histograms of this value with care
- Model performance decreases towards higher redshifts, mainly due to smaller training sample sizes
- The model expects 150 MHz radio data; 1.4 GHz data must be converted by the user, and this conversion reduces performance by ~4% in precision and ~3% in recall
- Wrongly cross-matched radio data or columns with very few samples can cause bad predictions; the algorithm is relatively new so issues are expected — report problems on GitHub
- The model was trained on three LOFAR Deep Fields (Lockman Hole, Boötes, ELAIS-N1); far-infrared and radio bands are of great importance, so missing those bands degrades classification

## Evidence

- [readme] README usage and run command: "python MBASC.py [path/to/file.csv]"
- [readme] Output file description: "The results.csv file has 3 columns: id, output, and AGN."
- [methods] Rounding rule: "This value is between 0 and 1 and is simply rounded to result in the AGN column."
- [readme] Output caveat: "the output column **does not** have to correspond to a likelihood that a source is an AGN"
- [readme] Unit requirement: "All units (except redshift) must be converted to μJy."
