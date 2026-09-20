---
name: agn-sfg-supervised-label-classification
description: Use when you have a photometric catalog with multi-band fluxes (optical
  u-r, NIR JHK, IRAC ch1-ch4, MIPS24, PACS/SPIRE far-IR, 150 MHz radio) and redshifts,
  and need AGN/SFG labels for extragalactic radio survey sources.
when_to_use_negative:
- The input is 1.4 GHz radio data that has not been converted to 150 MHz — misclassification
  is expected until conversion is done
- Flux units are not μJy — unit mismatch is listed as a leading cause of systematic
  misclassification in the README troubleshooting
- Key photometric columns contain very few samples or poorly cross-matched radio data
  — the model may make wrong predictions; such columns should be removed rather than
  passed in
edam_operation: http://edamontology.org/operation_2995
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: LightGBM
  role: Light gradient-boosting machine algorithm the supervised AGN/SFG classifier
    is based on
  repo: github.com/Jesper-Karsten/MBASC
- name: MBASC
  role: The packaged classifier pipeline run as `python MBASC.py [path/to/file.csv]`
    on the input CSV
  repo: github.com/Jesper-Karsten/MBASC
- name: pandas
  role: Required dependency for loading the input CSV of multi-band fluxes and redshifts
- name: sklearn
  role: Required dependency for train/test splitting and per-class precision/recall
    computation
- name: numpy
  role: Required dependency for array handling in the pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202346770
    title: A multi-band AGN-SFG classifier for extragalactic radio surveys using machine
      learning
  source_content_hash: sha256:e300713cd882da9c0cfa1b6189e199b9c9e572e85c7cf5efcdbc646126f8c186
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
  edam_operation: http://edamontology.org/operation_2995
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# agn-sfg-supervised-label-classification

## Summary

Supervised binary classification of radio continuum sources as AGNs versus star-forming galaxies (SFGs) using a LightGBM model trained on multi-wavelength photometry plus redshift from the three LOFAR Deep Fields, packaged as the MBASC tool.

## When to use

Use when you have a photometric catalog with multi-band fluxes (optical u-r, NIR JHK, IRAC ch1-ch4, MIPS24, PACS/SPIRE far-IR, 150 MHz radio) and redshifts, and need AGN/SFG labels for extragalactic radio survey sources.

## When NOT to use

- The input is 1.4 GHz radio data that has not been converted to 150 MHz — misclassification is expected until conversion is done
- Flux units are not μJy — unit mismatch is listed as a leading cause of systematic misclassification in the README troubleshooting
- Key photometric columns contain very few samples or poorly cross-matched radio data — the model may make wrong predictions; such columns should be removed rather than passed in

## Inputs

- CSV photometric catalog with exact headers: redshift, u, g, r, i, z, y, J, H, K, ch1-ch4 (IRAC 3.6/4.5/5.7/7.9 μm), MIPS24, PACS100, PACS160, SPIRE250, SPIRE350, SPIRE500, radiototal, radiopeak
- All flux densities converted to μJy; optional id column
- 150 MHz LOFAR radio fluxes (radiototal/radiopeak); 1.4 GHz data usable if converted to 150 MHz by the user

## Outputs

- output/results.csv with three columns: id, output (raw LightGBM value between 0 and 1), and AGN (rounded binary: 1 = AGN, 0 = SFG)
- Saved trained model file lgbm_agn_sfg_model.txt
- Per-class precision/recall metrics table (SFG: precision 0.92±0.01, recall 0.87±0.02; AGN: precision 0.87±0.02, recall 0.78±0.02)

## How to apply

Load the input CSV with pandas keeping the exact feature headers (redshift, u/g/r/i/z/y, J/H/K, ch1-ch4, MIPS24, PACS100/160, SPIRE250/350/500, radiototal, radiopeak) and convert every flux to μJy (redshift excluded). The LightGBM classifier was trained on SED-derived AGN/SFG labels from the Lockman Hole, Boötes, and ELAIS-N1 LOFAR Deep Fields; run it on the CSV, obtain the raw 0-1 output in output/results.csv, and round that value to produce the binary AGN column (1 = AGN, 0 = SFG). The raw output column does not have to correspond to a likelihood that a source is an AGN, so treat it cautiously. Missing features can simply be left out of the CSV header, and the model also works with 1.4 GHz radio data converted to 150 MHz (at a cost of ~4% precision and ~3% recall).

## Related tools

- **LightGBM** (Light gradient-boosting machine algorithm the supervised AGN/SFG classifier is based on) — github.com/Jesper-Karsten/MBASC
- **MBASC** (The packaged classifier pipeline run as `python MBASC.py [path/to/file.csv]` on the input CSV) — github.com/Jesper-Karsten/MBASC
- **pandas** (Required dependency for loading the input CSV of multi-band fluxes and redshifts)
- **sklearn** (Required dependency for train/test splitting and per-class precision/recall computation)
- **numpy** (Required dependency for array handling in the pipeline)

## Examples

```
python MBASC.py example.csv
```

## Evaluation signals

- results.csv must contain exactly three columns (id, output, AGN), be in the same row order as the input file, and the AGN column must equal the rounded output value
- Output values must lie in [0, 1] and be interpretable as a fraction, not a probability/likelihood of AGN status
- Benchmark performance: SFG precision 0.92±0.01 / recall 0.87±0.02; AGN precision 0.87±0.02 / recall 0.78±0.02; expect worse performance at higher redshifts due to smaller training sample sizes
- Running on the shipped example.csv (`python MBASC.py example.csv`) should complete and write output/results.csv
- If nearly all sources are classified to one class, suspect wrong units, wrong radio waveband, or sparse/poorly cross-matched feature columns

## Limitations

- The raw output column does not have to correspond to a likelihood that a source is an AGN; it depends on overall classifier quality and should be used cautiously
- Model performance decreases towards higher redshifts, mainly due to smaller training sample sizes
- Using 1.4 GHz data converted to 150 MHz reduces performance by ~4% in precision and ~3% in recall
- Far-infrared and radio bands are of great importance, so poorly cross-matched radio data or sparse columns can yield bad results
- 1.4 GHz to 150 MHz conversion must be done by the user; it is not performed by the model (may be added later)

## Evidence

- [abstract] Supervised LGBM classification on LOFAR Deep Fields: "We performed a supervised classification of SFGs versus AGNs using the light gradient boosting machine (LGBM) on three LOFAR Deep Fields (Lockman Hole, Boötes, and ELAIS-N1)"
- [methods] Round raw output to binary AGN column: "This value is between 0 and 1 and is simply rounded to result in the AGN column. The AGN column is 1 for a predicted AGN and 0 for a predicted SFG."
- [readme] Output is not an AGN likelihood: "the output column **does not** have to correspond to a likelihood that a source is an AGN"
- [readme] Unit conversion requirement: "If a certain feature is not available, it can simply be left out. All units (except redshift) must be converted to μJy."
- [abstract] Performance metrics: "Our trained model has a precision of 0.92±0.01 and a recall of 0.87±0.02 for SFGs. For AGNs, the model performs slightly worse, with a precision of 0.87±0.02 and a recall of 0.78±0.02."
- [abstract] 1.4 GHz conversion penalty: "Converting 1.4 GHz to 150 MHz radio data reduces the performance by ~4% in precision and ~3% in recall."
