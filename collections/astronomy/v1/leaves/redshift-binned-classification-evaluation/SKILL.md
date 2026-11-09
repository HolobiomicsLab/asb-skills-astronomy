---
name: redshift-binned-classification-evaluation
description: Use when you have classifier predictions plus true labels and redshifts
  for a test set and need to quantify how classification performance (precision/recall)
  varies across redshift bins rather than only over the full sample.
when_to_use_negative:
- Input fluxes are in units other than μJy or the radio data is 1.4 GHz rather than
  150 MHz — the README warns misclassified-heavy results follow from unconverted units
  or wrong radio band
- The task is to use the raw output column as a probability — the README explicitly
  states it does not have to correspond to a likelihood that a source is an AGN
- You need an overall performance estimate only, with no redshift-resolved breakdown
  — a single confusion matrix over the full test set suffices
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: MBASC
  role: Runs the LightGBM AGN/SFG classifier whose per-redshift-bin performance is
    being evaluated
  repo: github.com/Jesper-Karsten/MBASC
- name: LightGBM
  role: Underlying light gradient boosting machine technique producing the raw classification
    scores
- name: sklearn
  role: Computes per-bin precision and recall against true labels
- name: pandas
  role: Loads output/results.csv and groups test-set sources into redshift bins
- name: numpy
  role: Dependency for the numerical binning and metric computation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202346770
    title: A multi-band AGN-SFG classifier for extragalactic radio surveys using machine
      learning
  source_content_hash: sha256:69871437f5a3c08f4fc84775552f903b3cf308100fce90ecd24eb5a0910efa72
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# redshift-binned-classification-evaluation

## Summary

Evaluates the MBASC LightGBM AGN/SFG classifier by binning test-set sources into redshift intervals and computing per-bin precision and recall, revealing performance degradation towards higher redshifts attributed mainly to smaller training sample sizes at high z.

## When to use

Use when you have classifier predictions plus true labels and redshifts for a test set and need to quantify how classification performance (precision/recall) varies across redshift bins rather than only over the full sample.

## When NOT to use

- Input fluxes are in units other than μJy or the radio data is 1.4 GHz rather than 150 MHz — the README warns misclassified-heavy results follow from unconverted units or wrong radio band
- The task is to use the raw output column as a probability — the README explicitly states it does not have to correspond to a likelihood that a source is an AGN
- You need an overall performance estimate only, with no redshift-resolved breakdown — a single confusion matrix over the full test set suffices

## Inputs

- Labelled test-set CSV with multi-wavelength flux densities (u, g, r, i, z, y, J, H, K, ch1-ch4, MIPS24, PACS100/160, SPIRE250/350/500, radiototal, radiopeak) converted to μJy, plus redshift column and true AGN/SFG labels
- output/results.csv from MBASC.py containing id, raw LightGBM output (0-1), and rounded AGN column

## Outputs

- Per-redshift-bin precision and recall tables for SFG and AGN classes computed with sklearn
- Full-test-set precision/recall summary (target: SFG precision 0.92±0.01, recall 0.87±0.02; AGN precision 0.87±0.02, recall 0.78±0.02)
- Confirmed trend of decreasing performance towards higher redshifts

## How to apply

First run the MBASC LightGBM classifier (python MBASC.py) on a labelled CSV whose multi-wavelength fluxes are converted to μJy and which includes redshifts, producing output/results.csv with id, output (raw LightGBM score between 0 and 1), and AGN columns. Round the raw output to obtain the binary AGN column (1 = predicted AGN, 0 = predicted SFG), which may not correspond to a likelihood that a source is an AGN. Bin the test-set sources by redshift into consecutive redshift intervals, then use sklearn metrics to compute per-bin precision and recall for the SFG and AGN classes against the true labels. Tabulate the per-bin metrics and inspect the trend: the expected behaviour is an overall decrease in performance towards higher-redshift bins, which the paper attributes mainly to smaller training sample sizes at high z. Also verify the full-test-set metrics match the reported values (SFG precision 0.92±0.01, recall 0.87±0.02; AGN precision 0.87±0.02, recall 0.78±0.02).

## Related tools

- **MBASC** (Runs the LightGBM AGN/SFG classifier whose per-redshift-bin performance is being evaluated) — github.com/Jesper-Karsten/MBASC
- **LightGBM** (Underlying light gradient boosting machine technique producing the raw classification scores)
- **sklearn** (Computes per-bin precision and recall against true labels)
- **pandas** (Loads output/results.csv and groups test-set sources into redshift bins)
- **numpy** (Dependency for the numerical binning and metric computation)

## Examples

```
python MBASC.py example.csv; import pandas as pd; from sklearn.metrics import precision_score, recall_score; df = pd.read_csv('output/results.csv')
```

## Evaluation signals

- Per-bin precision/recall for SFG and AGN classes show a monotonic or overall decreasing trend towards higher redshift bins
- Full-test-set metrics are consistent with reported values: SFG precision 0.92±0.01 and recall 0.87±0.02; AGN precision 0.87±0.02 and recall 0.78±0.02
- AGN column in results.csv is binary (0/1) obtained by rounding the raw output which lies between 0 and 1
- Rows of results.csv remain in the same order as the input file so labels can be joined by position or id
- All fluxes were verified as converted to μJy before classification, since unit errors distort classification and thus the binned metrics

## Limitations

- The raw output column does not have to correspond to a likelihood that a source is an AGN; treat it with care (e.g. only analyse histograms cautiously)
- Performance degradation at high redshift is attributed mainly to smaller training sample sizes, so per-bin metrics at high z rest on few sources
- The model expects 150 MHz radio data; using 1.4 GHz data or converting from 1.4 GHz degrades performance (~4% precision, ~3% recall loss for converted data)

## Evidence

- [abstract] The model performance decreases towards higher redshifts, which is mainly due to smaller training sample sizes.: "The model performance decreases towards higher redshifts, which is mainly due to smaller training sample sizes."
- [abstract] Trained-model metric targets: "Our trained model has a precision of 0.92±0.01 and a recall of 0.87±0.02 for SFGs. For AGNs, the model performs slightly worse, with a precision of 0.87±0.02 and a recall of 0.78±0.02."
- [readme] Output interpretation: "This value is between 0 and 1 and is simply rounded to result in the AGN column. The AGN column is 1 for a predicted AGN and 0 for a predicted SFG."
- [readme] Output column caveat: "the output column **does not** have to correspond to a likelihood that a source is an AGN"
- [readme] Unit requirement: "All units (except redshift) must be converted to μJy."
