---
name: precision-recall-metric-evaluation
description: Use when a supervised binary classifier (e.g. AGN=1 vs SFG=0 from rounded
  LightGBM output in output/results.csv) must be scored per class with precision/recall
  and validated against published reference metrics.
when_to_use_negative:
- The raw 'output' column is needed as a confidence/likelihood — the README states
  it does not have to correspond to a likelihood that a source is an AGN
- The task is multi-class or probabilistic calibration rather than binary AGN/SFG
  classification with rounded LightGBM output
- Input labels are not SED-derived AGN/SFG classifications, so per-class precision/recall
  against reference metrics is not meaningful
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3315
tools:
- name: MBASC
  role: LightGBM-based AGN/SFG classifier whose predictions are scored; run as 'python
    MBASC.py [path/to/file.csv]'
  repo: https://github.com/Jesper-Karsten/MBASC
- name: sklearn
  role: Computes per-class precision and recall against held-out SED-derived labels
- name: pandas
  role: Loads the held-out catalog and output/results.csv, verifying column headers
    and μJy flux units
- name: lightgbm
  role: Produces the raw 0–1 output fraction that is rounded to the binary AGN column
    before metric computation
- name: numpy
  role: Supports flux conversion to μJy and perturbation of photometric fluxes for
    SNR/band-degradation experiments
provenance:
  source_task_ids:
  - task_002
  - task_006
  - task_007
  - task_001
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202346770
    title: A multi-band AGN-SFG classifier for extragalactic radio surveys using machine
      learning
  source_content_hash: sha256:f3317cf1d6b986d0a3e89447c586f316b69c96ec0da19474a7268f32f85f7583
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
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# precision-recall-metric-evaluation

## Summary

Evaluate a binary classifier's per-class precision and recall (here, MBASC's LightGBM AGN-vs-SFG classifier on LOFAR Deep Fields data) using sklearn against held-out SED-derived labels, and compare the metrics against reference values (SFG precision 0.92±0.01, recall 0.87±0.02; AGN precision 0.87±0.02, recall 0.78±0.02).

## When to use

Use when a supervised binary classifier (e.g. AGN=1 vs SFG=0 from rounded LightGBM output in output/results.csv) must be scored per class with precision/recall and validated against published reference metrics.

## When NOT to use

- The raw 'output' column is needed as a confidence/likelihood — the README states it does not have to correspond to a likelihood that a source is an AGN
- The task is multi-class or probabilistic calibration rather than binary AGN/SFG classification with rounded LightGBM output
- Input labels are not SED-derived AGN/SFG classifications, so per-class precision/recall against reference metrics is not meaningful

## Inputs

- Held-out catalog CSV with MBASC feature headers (redshift, u, g, r, i, z, y, J, H, K, ch1–ch4, MIPS24, PACS100/160, SPIRE250/350/500, radiototal, radiopeak), fluxes in μJy
- Held-out AGN/SFG labels derived from SED analyses
- output/results.csv from MBASC containing id, output (raw 0–1 fraction), and AGN (binary) columns

## Outputs

- Per-class precision and recall table (CSV) for SFGs and AGNs
- Metric deltas versus reference values or baseline configuration (e.g. band_degradation_performance.csv, per-SNR metrics table)

## How to apply

Run the classifier (e.g. 'python MBASC.py [path/to/file.csv]'), read output/results.csv, and round the raw LightGBM fraction (0–1) to obtain the binary AGN column (1 = AGN, 0 = SFG); note the raw output is not a likelihood of being an AGN. Compute per-class precision and recall with sklearn on the held-out SED-derived labels, treating SFG and AGN as separate classes. Compare the computed metrics to the reference values: SFG precision 0.92±0.01 and recall 0.87±0.02, AGN precision 0.87±0.02 and recall 0.78±0.02. For perturbation experiments (e.g. 1.4 GHz radio fluxes converted to 150 MHz, degraded SNR photometry, or dropped bands), record metric deltas against the native 150 MHz full-band baseline. Save the per-class scores as a metrics table alongside results.csv.

## Related tools

- **MBASC** (LightGBM-based AGN/SFG classifier whose predictions are scored; run as 'python MBASC.py [path/to/file.csv]') — https://github.com/Jesper-Karsten/MBASC
- **sklearn** (Computes per-class precision and recall against held-out SED-derived labels)
- **pandas** (Loads the held-out catalog and output/results.csv, verifying column headers and μJy flux units)
- **lightgbm** (Produces the raw 0–1 output fraction that is rounded to the binary AGN column before metric computation)
- **numpy** (Supports flux conversion to μJy and perturbation of photometric fluxes for SNR/band-degradation experiments)

## Examples

```
python MBASC.py example.csv; import pandas as pd, sklearn.metrics; df = pd.read_csv('output/results.csv'); sklearn.metrics.precision_recall_fscore_support(labels, df['AGN'], average=None)
```

## Evaluation signals

- Computed SFG precision falls within 0.92±0.01 and SFG recall within 0.87±0.02
- Computed AGN precision falls within 0.87±0.02 and AGN recall within 0.78±0.02
- results.csv contains the three columns id, output, and AGN, in the same order as the input file, with AGN = round(output) restricted to {0, 1}
- For the 1.4 GHz→150 MHz conversion experiment, precision drops ~4% and recall drops ~3% relative to native 150 MHz data
- Band-removal degradation shows the largest metric drops when far-infrared and radio bands are removed

## Limitations

- The raw output column does not correspond to a likelihood that a source is an AGN; it depends on classifier performance on the whole dataset, so histogram-based interpretation requires care
- Model performance decreases towards higher redshifts, mainly due to smaller training sample sizes
- Using 1.4 GHz radio data without converting to 150 MHz, or wrong μJy units, or wrongly cross-matched radio data can cause systematic misclassifications that invalidate the metrics
- Sparse columns with very few samples can lead the model to make wrong predictions; removing such columns is the documented remedy

## Evidence

- [abstract] reference metrics: "Our trained model has a precision of 0.92±0.01 and a recall of 0.87±0.02 for SFGs. For AGNs, the model performs slightly worse, with a precision of 0.87±0.02 and a recall of 0.78±0.02."
- [readme] rounding rule: "This value is between 0 and 1 and is simply rounded to result in the AGN column. The AGN column is 1 for a predicted AGN and 0 for a predicted SFG."
- [readme] output caveat: "the output column **does not** have to correspond to a likelihood that a source is an AGN"
- [abstract] conversion performance delta: "Converting 1.4 GHz to 150 MHz radio data reduces the performance by ~4% in precision and ~3% in recall."
- [readme] sparse-column failure mode: "Remove data columns with very few samples, the model might make wrong predictions based on this."
