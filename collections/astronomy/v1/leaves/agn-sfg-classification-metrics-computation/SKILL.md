---
name: agn-sfg-classification-metrics-computation
description: Use when you have run MBASC on a labelled test set with multi-wavelength
  fluxes (μJy) and redshifts, and need to evaluate how SFG/AGN precision and recall
  vary across redshift bins rather than only on the full test set.
when_to_use_negative:
- You only need full-test-set aggregate metrics (SFG precision 0.92±0.01, recall 0.87±0.02;
  AGN precision 0.87±0.02, recall 0.78±0.02) rather than their redshift dependence.
- Input radio data is 1.4 GHz rather than 150 MHz — convert or note the ~4% precision
  / ~3% recall penalty before evaluating performance.
- The raw output column is needed as a probability/likelihood of being an AGN; the
  README warns it does not correspond to a likelihood.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: MBASC
  role: LightGBM-based AGN/SFG classifier whose results.csv output provides the predictions
    to be binned and scored
  repo: github.com/Jesper-Karsten/MBASC
- name: pandas
  role: Load and bin results.csv and the labelled test set by redshift
- name: sklearn
  role: Compute per-bin precision and recall for SFG and AGN classes
- name: lightgbm
  role: Underlying gradient-boosting model producing the raw output score rounded
    into the AGN column
- name: numpy
  role: Numerical handling of redshift bins and metric arrays
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202346770
    title: A multi-band AGN-SFG classifier for extragalactic radio surveys using machine
      learning
  source_content_hash: sha256:b023603dae9129c68eb94e5082360346c96dede7cb90f8042ee4bca304fa52f9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MBASC
  - Python
  - lightgbm
  - numpy
  - pandas
  - sklearn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# agn-sfg-classification-metrics-computation

## Summary

Computes per-redshift-bin precision and recall for the SFG and AGN classes of the LGBM-based MBASC classifier, quantifying how classification performance degrades towards higher redshifts due to smaller training sample sizes.

## When to use

Use when you have run MBASC on a labelled test set with multi-wavelength fluxes (μJy) and redshifts, and need to evaluate how SFG/AGN precision and recall vary across redshift bins rather than only on the full test set.

## When NOT to use

- You only need full-test-set aggregate metrics (SFG precision 0.92±0.01, recall 0.87±0.02; AGN precision 0.87±0.02, recall 0.78±0.02) rather than their redshift dependence.
- Input radio data is 1.4 GHz rather than 150 MHz — convert or note the ~4% precision / ~3% recall penalty before evaluating performance.
- The raw output column is needed as a probability/likelihood of being an AGN; the README warns it does not correspond to a likelihood.

## Inputs

- output/results.csv from MBASC with columns id, output (raw LightGBM score between 0 and 1), and AGN (0/1)
- Labelled test-set CSV of multi-wavelength flux densities (u through K bands, IRAC ch1-ch4, MIPS24, PACS100/160, SPIRE250/350/500, 150 MHz radiototal/radiopeak) converted to μJy, plus redshift and true AGN/SFG labels

## Outputs

- Per-redshift-bin precision and recall tables for the SFG and AGN classes
- Trend assessment of classification performance with increasing redshift

## How to apply

Run the MBASC LightGBM classifier (python MBASC.py [path/to/file.csv]) on the test-set CSV with all fluxes converted to μJy plus redshifts; it writes id, output, and AGN columns to output/results.csv. Load results.csv with pandas and treat the rounded AGN column (0/1, obtained by rounding the raw LightGBM output between 0 and 1) as the predicted class. Bin test-set sources by redshift into consecutive redshift intervals, then use sklearn metrics to compute per-bin precision and recall for the SFG and AGN classes against the true labels. Tabulate the per-bin metrics and inspect the trend with redshift, confirming whether performance decreases towards higher redshifts, which the paper attributes mainly to smaller training sample sizes at high z. Note that the raw output column does not have to correspond to a likelihood that a source is an AGN, so avoid treating it as a calibrated probability when binning.

## Related tools

- **MBASC** (LightGBM-based AGN/SFG classifier whose results.csv output provides the predictions to be binned and scored) — github.com/Jesper-Karsten/MBASC
- **pandas** (Load and bin results.csv and the labelled test set by redshift)
- **sklearn** (Compute per-bin precision and recall for SFG and AGN classes)
- **lightgbm** (Underlying gradient-boosting model producing the raw output score rounded into the AGN column)
- **numpy** (Numerical handling of redshift bins and metric arrays)

## Examples

```
python MBASC.py example.csv; import pandas as pd; df = pd.read_csv('output/results.csv'); pred = df['AGN'].round(); from sklearn.metrics import precision_score, recall_score
```

## Evaluation signals

- Per-bin precision/recall values show a monotonic or overall decreasing trend towards higher redshift bins.
- Full-test-set metrics are consistent with reported values: SFG precision 0.92±0.01 and recall 0.87±0.02; AGN precision 0.87±0.02 and recall 0.78±0.02.
- results.csv retains the same source order as the input file and the AGN column contains only 0/1 values after rounding the raw output.
- Redshift bins are consecutive and cover the test set without dropping sources.

## Limitations

- The raw output column does not have to correspond to a likelihood that a source is an AGN; interpret histograms of this value with care.
- Performance degradation at high redshift is attributed to smaller training sample sizes, so per-bin metrics at high z rest on few sources and are correspondingly noisier.
- Metric quality depends on correct input preparation: all units must be μJy, radio data must be 150 MHz (not 1.4 GHz), and sparsely sampled columns or wrongly cross-matched radio data can cause systematic misclassifications.

## Evidence

- [abstract] Performance decreases with redshift: "The model performance decreases towards higher redshifts, which is mainly due to smaller training sample sizes."
- [abstract] Reported test-set metrics: "Our trained model has a precision of 0.92±0.01 and a recall of 0.87±0.02 for SFGs. For AGNs, the model performs slightly worse, with a precision of 0.87±0.02 and a recall of 0.78±0.02."
- [readme] Rounding of raw output to binary class: "This value is between 0 and 1 and is simply rounded to result in the AGN column. The AGN column is 1 for a predicted AGN and 0 for a predicted SFG."
- [readme] Raw output is not a probability: "the output column **does not** have to correspond to a likelihood that a source is an AGN"
- [methods] Output file location: "Output of the model will be in output/results.csv."
