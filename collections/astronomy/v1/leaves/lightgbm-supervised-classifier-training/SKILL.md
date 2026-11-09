---
name: lightgbm-supervised-classifier-training
description: Use when you have a CSV of multi-wavelength flux densities in μJy with
  redshift and need a supervised SFG-vs-AGN classification, or must retrain the classifier
  on degraded/modified band subsets to probe feature importance.
when_to_use_negative:
- Your labels are not binary SFG-vs-AGN classes (e.g. you need AGN sub-type or radio-AGN
  classification) — MBASC is specifically an SFG-versus-AGN classifier
- You need the raw output column as a probability or likelihood of being an AGN —
  the README explicitly warns it 'does not have to correspond to a likelihood'
- Your fluxes are not in μJy and cannot be converted — unit mismatch is a documented
  cause of wholesale misclassification
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: LightGBM
  role: Gradient-boosting machine algorithm underlying the SFG-vs-AGN supervised classifier
  repo: github.com/Jesper-Karsten/MBASC
- name: MBASC
  role: Wrapper script (MBASC.py) that loads the input CSV, runs the LightGBM model,
    and writes results.csv
  repo: github.com/Jesper-Karsten/MBASC
- name: pandas
  role: Dependency for reading and preparing the input feature CSV
- name: sklearn
  role: Dependency used for evaluating classifier precision and recall per class
- name: numpy
  role: Dependency for numerical handling of flux features
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202346770
    title: A multi-band AGN-SFG classifier for extragalactic radio surveys using machine
      learning
  source_content_hash: sha256:5ba4f707927a5a1f1ce6243a277e5ecfcab1405555114fda7e8341e471b58c7b
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# lightgbm-supervised-classifier-training

## Summary

Trains a LightGBM (LGBM) supervised classifier to separate AGNs from star-forming galaxies (SFGs) using multi-wavelength photometry (u through SPIRE500, radiototal/radiopeak) plus redshift, as implemented in the MBASC pipeline for LOFAR Deep Fields data.

## When to use

Use when you have a CSV of multi-wavelength flux densities in μJy with redshift and need a supervised SFG-vs-AGN classification, or must retrain the classifier on degraded/modified band subsets to probe feature importance.

## When NOT to use

- Your labels are not binary SFG-vs-AGN classes (e.g. you need AGN sub-type or radio-AGN classification) — MBASC is specifically an SFG-versus-AGN classifier
- You need the raw output column as a probability or likelihood of being an AGN — the README explicitly warns it 'does not have to correspond to a likelihood'
- Your fluxes are not in μJy and cannot be converted — unit mismatch is a documented cause of wholesale misclassification

## Inputs

- CSV file of flux densities in μJy with headers matching MBASC feature names exactly (e.g. example.csv in the MBASC repo)
- Redshift column for every source
- Optional id column for preserving source identifiers
- Optionally, 1.4 GHz radio flux densities manually converted to 150 MHz-equivalent radio data

## Outputs

- output/results.csv with columns: id, output (raw LightGBM fraction 0–1), and AGN (binary, rounded)
- Per-class precision/recall metrics for SFGs and AGNs (baseline: SFG precision 0.92±0.01, recall 0.87±0.02; AGN precision 0.87±0.02, recall 0.78±0.02)

## How to apply

Prepare an input CSV whose headers exactly match MBASC feature names (u, g, r, i, z, y, J, H, K, ch1-ch4, MIPS24, PACS100/160, SPIRE250/350/500, radiototal, radiopeak, redshift), converting every flux density to μJy; unavailable features may simply be omitted, which is also how degraded band configurations are created. Run the LightGBM classifier via MBASC.py, keeping redshift even when photometric columns are dropped. The model emits a raw fractional output between 0 and 1 per source; round this to form the binary AGN column (1 = AGN, 0 = SFG). Evaluate on held-out data with sklearn, computing precision and recall separately per class, and expect degradation experiments to show the largest performance drops when far-infrared (PACS/SPIRE) and radio bands are removed.

## Related tools

- **LightGBM** (Gradient-boosting machine algorithm underlying the SFG-vs-AGN supervised classifier) — github.com/Jesper-Karsten/MBASC
- **MBASC** (Wrapper script (MBASC.py) that loads the input CSV, runs the LightGBM model, and writes results.csv) — github.com/Jesper-Karsten/MBASC
- **pandas** (Dependency for reading and preparing the input feature CSV)
- **sklearn** (Dependency used for evaluating classifier precision and recall per class)
- **numpy** (Dependency for numerical handling of flux features)

## Examples

```
python MBASC.py example.csv
```

## Evaluation signals

- All input flux columns are in μJy and column headers match MBASC feature names exactly (case-insensitive), otherwise wavebands are silently unrecognized
- Full-band baseline reproduces reported performance: precision 0.92±0.01 / recall 0.87±0.02 for SFGs and precision 0.87±0.02 / recall 0.78±0.02 for AGNs
- output/results.csv contains exactly three columns (id, output, AGN), in the same row order as the input file, with AGN values in {0, 1}
- Band-degradation experiments show the largest precision/recall drops when far-infrared and radio bands are removed, consistent with reported feature importance
- Predictions are not degenerate (all AGN or all SFG) — a documented symptom of unit errors or sparse columns; removing data columns with very few samples is the documented remedy

## Limitations

- The raw output column is a fraction between 0 and 1 but is NOT a likelihood of being AGN; its interpretability depends on overall classifier quality
- Model performance decreases towards higher redshifts, mainly due to smaller training sample sizes
- 1.4 GHz radio data must be manually converted before use; using it unconverted (or other wavebands instead of 150 MHz) causes bad predictions — converting 1.4 GHz to 150 MHz reduces performance by ~4% in precision and ~3% in recall
- Wrongly cross-matched radio data can give bad results; the README suggests removing or dropping problematic features (particularly radio data) when misclassifications occur

## Evidence

- [abstract] We performed a supervised classification of SFGs versus AGNs using the light gradient boosting machine (LGBM) on three LOFAR Deep Fields (Lockman Hole, Boötes, and ELAIS-N1): "We performed a supervised classification of SFGs versus AGNs using the light gradient boosting machine (LGBM) on three LOFAR Deep Fields (Lockman Hole, Boötes, and ELAIS-N1)"
- [abstract] Reported class-wise performance metrics: "Our trained model has a precision of 0.92±0.01 and a recall of 0.87±0.02 for SFGs. For AGNs, the model performs slightly worse, with a precision of 0.87±0.02 and a recall of 0.78±0.02."
- [abstract] Feature importance of far-IR and radio bands: "we find that the far-infrared and radio bands are of great importance."
- [readme] Unit requirement for input features: "All units (except redshift) must be converted to μJy."
- [readme] Output schema and rounding rule: "This value is between 0 and 1 and is simply rounded to result in the AGN column. The AGN column is 1 for a predicted AGN and 0 for a predicted SFG."
- [readme] Raw output is not a probability: "the output column **does not** have to correspond to a likelihood that a source is an AGN"
