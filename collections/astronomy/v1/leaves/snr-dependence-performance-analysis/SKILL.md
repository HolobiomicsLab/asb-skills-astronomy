---
name: snr-dependence-performance-analysis
description: Use when you need to test whether higher SNR in some photometric bands
  yields a significant boost in MBASC (AGN vs SFG) classification performance, e.g.
  before deploying the classifier on shallower or noisier survey data.
when_to_use_negative:
- You only need a single classification run on clean data — run MBASC directly instead
  of a perturbation sweep
- Your goal is feature-importance ranking (which bands matter) rather than SNR sensitivity
  — the paper addresses band importance separately
- Your data uses 1.4 GHz or other non-150 MHz radio fluxes without conversion — the
  README warns this causes wrong classifications
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: MBASC
  role: LightGBM-based AGN/SFG classifier run on each perturbed CSV
  repo: https://github.com/Jesper-Karsten/MBASC
- name: pandas
  role: Loading the input catalog and assembling the per-SNR metrics table
- name: numpy
  role: Adding calibrated noise to band flux densities to generate SNR levels
- name: sklearn
  role: Computing precision and recall per SNR level against reference classifications
- name: lightgbm
  role: Underlying model dependency of MBASC producing the raw fractional output
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1051/0004-6361/202346770
    title: A multi-band AGN-SFG classifier for extragalactic radio surveys using machine
      learning
  source_content_hash: sha256:e1d471e5d5ccbe9e5aee6ec2f329dc74526196f5a48462a0ea6c81aca0bc41d5
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
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# snr-dependence-performance-analysis

## Summary

Quantifies how photometric-band signal-to-noise ratio affects MBASC classifier performance by perturbing band flux densities (u through SPIRE500) with calibrated noise, re-running the LightGBM-based classifier at each SNR level, and assembling per-level precision/recall metrics.

## When to use

Use when you need to test whether higher SNR in some photometric bands yields a significant boost in MBASC (AGN vs SFG) classification performance, e.g. before deploying the classifier on shallower or noisier survey data.

## When NOT to use

- You only need a single classification run on clean data — run MBASC directly instead of a perturbation sweep
- Your goal is feature-importance ranking (which bands matter) rather than SNR sensitivity — the paper addresses band importance separately
- Your data uses 1.4 GHz or other non-150 MHz radio fluxes without conversion — the README warns this causes wrong classifications

## Inputs

- Input catalog CSV (e.g. example.csv from the MBASC repository) with flux densities in μJy and headers matching MBASC feature names (redshift, u, g, r, i, z, y, J, H, K, ch1-ch4, MIPS24, PACS100, PACS160, SPIRE250, SPIRE350, SPIRE500, radiototal, radiopeak)
- Reference (ground-truth) AGN/SFG classifications for the sources

## Outputs

- One perturbed CSV per SNR level containing the same sources with noise-degraded flux densities
- Raw LightGBM outputs per SNR level (output/results.csv with id, output, AGN columns)
- Single metrics table CSV with per-SNR precision and recall for SFGs and AGNs, ordered lowest to highest SNR

## How to apply

Load the input catalog (e.g. example.csv from the MBASC repository) with pandas, ensuring all flux densities are in μJy and headers match the required feature names exactly. Use numpy to perturb the photometric band flux densities (u through SPIRE500) with calibrated noise so the same sources are represented at a series of decreasing SNR levels, writing one modified CSV per SNR level. Run the LightGBM-based MBASC algorithm on each perturbed CSV via 'python MBASC.py <file.csv>', collecting the raw fractional LightGBM output written to output/results.csv, then round it to the binary AGN column (1 = AGN, 0 = SFG) as MBASC specifies. Compute precision and recall for SFGs and AGNs at each SNR level with sklearn against reference classifications, replicating the metric definitions behind the reported 0.92/0.87 SFG and 0.87/0.78 AGN performance, and assemble a single metrics table CSV ordered from lowest to highest SNR.

## Related tools

- **MBASC** (LightGBM-based AGN/SFG classifier run on each perturbed CSV) — https://github.com/Jesper-Karsten/MBASC
- **pandas** (Loading the input catalog and assembling the per-SNR metrics table)
- **numpy** (Adding calibrated noise to band flux densities to generate SNR levels)
- **sklearn** (Computing precision and recall per SNR level against reference classifications)
- **lightgbm** (Underlying model dependency of MBASC producing the raw fractional output)

## Examples

```
python MBASC.py example.csv
```

## Evaluation signals

- The metrics table CSV exists and is valid with precision and recall columns per SNR level
- The table shows the reported trend: a higher signal-to-noise ratio in some photometric bands leads to a significant boost in model performance
- Baseline metrics at the unperturbed SNR are consistent with the paper's 0.92±0.01/0.87±0.02 SFG and 0.87±0.02/0.78±0.02 AGN precision/recall
- Perturbed CSVs preserve the original sources and headers, with flux densities still in μJy and SNR monotonically decreasing across the series
- Each MBASC run produced output/results.csv with id, output, and AGN columns in the same order as the input file

## Limitations

- The raw LightGBM output column does not have to correspond to a likelihood that a source is an AGN — treat histograms of it cautiously
- Model performance decreases towards higher redshifts, mainly due to smaller training sample sizes, which can confound SNR trends
- Far-infrared and radio bands are of great importance for classification; degrading them may dominate the SNR trend
- Removing data columns with very few samples is recommended, as the model might make wrong predictions based on sparse features
- Wrongly cross-matched radio data can give bad results

## Evidence

- [abstract] a higher signal-to-noise ratio in some photometric bands leads to a significant boost in the model performance.: "a higher signal-to-noise ratio in some photometric bands leads to a significant boost in the model performance."
- [readme] This value is between 0 and 1 and is simply rounded to result in the AGN column.: "This value is between 0 and 1 and is simply rounded to result in the AGN column. The AGN column is 1 for a predicted AGN and 0 for a predicted SFG."
- [readme] All units (except redshift) must be converted to μJy.: "If a certain feature is not available, it can simply be left out. All units (except redshift) must be converted to μJy."
- [readme] Output of the model will be in output/results.csv.: "Output of the model will be in output/results.csv."
- [abstract] Model performance decreases towards higher redshifts, mainly due to smaller training sample sizes: "The model performance decreases towards higher redshifts, which is mainly due to smaller training sample sizes."
