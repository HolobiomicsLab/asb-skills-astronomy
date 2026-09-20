---
name: multiwavelength-photometry-band-subset-degradation-analysis
description: Use when planning or interpreting radio galaxy surveys with poorer multi-wavelength
  coverage than LOFAR Deep Fields, and you need to know which photometric bands (especially
  far-infrared and radio) the classifier depends on.
when_to_use_negative:
- Your input is not a multi-wavelength photometric feature table with MBASC-compatible
  band columns (e.g. spectra-only or image data).
- You have 1.4 GHz radio data and cannot convert it to 150 MHz — the README warns
  misclassifications occur if 1.4 GHz or other wavebands are used instead of 150 MHz
  radio data.
- 'You need a probabilistic AGN score: the raw LightGBM output column does not have
  to correspond to a likelihood that a source is an AGN.'
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MBASC
  role: The multi-band AGN-SFG classifier whose band subsets are trained and evaluated
    in the degradation analysis
  repo: github.com/Jesper-Karsten/MBASC
- name: lightgbm
  role: Light gradient boosting machine used to train the classifier on each band
    subset
  repo: github.com/Jesper-Karsten/MBASC
- name: pandas
  role: Loads the LOFAR Deep Fields catalogs and constructs degraded band configurations
    by dropping columns
  repo: github.com/Jesper-Karsten/MBASC
- name: sklearn
  role: Computes class-specific precision and recall on held-out data for each degraded
    configuration
  repo: github.com/Jesper-Karsten/MBASC
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202346770
    title: A multi-band AGN-SFG classifier for extragalactic radio surveys using machine
      learning
  source_content_hash: sha256:80fcf2136bcf0a19541172b6715a59f2d5f0f5506e5b27c07183406c237e808a
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# multiwavelength-photometry-band-subset-degradation-analysis

## Summary

Systematically drops bands (u, g, r, i, z, y, J, H, K, ch1–ch4, MIPS24, PACS100/160, SPIRE250/350/500, radiototal, radiopeak) from the MBASC LightGBM feature set to measure how degraded SED sampling affects SFG-vs-AGN classification, identifying which bands matter most.

## When to use

Use when planning or interpreting radio galaxy surveys with poorer multi-wavelength coverage than LOFAR Deep Fields, and you need to know which photometric bands (especially far-infrared and radio) the classifier depends on.

## When NOT to use

- Your input is not a multi-wavelength photometric feature table with MBASC-compatible band columns (e.g. spectra-only or image data).
- You have 1.4 GHz radio data and cannot convert it to 150 MHz — the README warns misclassifications occur if 1.4 GHz or other wavebands are used instead of 150 MHz radio data.
- You need a probabilistic AGN score: the raw LightGBM output column does not have to correspond to a likelihood that a source is an AGN.

## Inputs

- LOFAR Deep Fields multi-wavelength catalogs (Lockman Hole, Boötes, ELAIS-N1) as CSV with MBASC feature names as headers (u, g, r, i, z, y, J, H, K, ch1-ch4, MIPS24, PACS100, PACS160, SPIRE250, SPIRE350, SPIRE500, radiototal, radiopeak, redshift)
- Flux density columns in μJy (all units except redshift)
- Rounded binary AGN column for class-level metric computation

## Outputs

- band_degradation_performance.csv: precision and recall per SFG and AGN class for each degraded band configuration
- Ranking of band importance showing which bands cause the largest performance drops when removed

## How to apply

Load the LOFAR Deep Fields multi-wavelength catalogs (Lockman Hole, Boötes, ELAIS-N1) with pandas, converting all flux densities to μJy and matching MBASC header names exactly. Generate degraded configurations by dropping columns from the full feature set (redshift kept), and additionally test configurations where sparse columns are removed — the documented remedy for misclassifications. Train the LightGBM (LGBM) classifier on each band subset following the MBASC supervised SFG-vs-AGN setup, evaluate on held-out data with sklearn, and compute precision and recall separately for SFGs and AGNs from the rounded binary AGN column. Assemble per-configuration metrics into a single table (e.g. band_degradation_performance.csv) and compare against the full-band baseline.

## Related tools

- **MBASC** (The multi-band AGN-SFG classifier whose band subsets are trained and evaluated in the degradation analysis) — github.com/Jesper-Karsten/MBASC
- **lightgbm** (Light gradient boosting machine used to train the classifier on each band subset) — github.com/Jesper-Karsten/MBASC
- **pandas** (Loads the LOFAR Deep Fields catalogs and constructs degraded band configurations by dropping columns) — github.com/Jesper-Karsten/MBASC
- **sklearn** (Computes class-specific precision and recall on held-out data for each degraded configuration) — github.com/Jesper-Karsten/MBASC

## Examples

```
python MBASC.py example.csv
```

## Evaluation signals

- Full-band baseline must reproduce precision 0.92±0.01 and recall 0.87±0.02 for SFGs and precision 0.87±0.02 and recall 0.78±0.02 for AGNs.
- Removing far-infrared (PACS100/160, SPIRE250/350/500) and radio (radiototal, radiopeak) bands should produce the largest precision/recall drops, consistent with their reported high importance.
- All flux columns in every configuration must be in μJy and headers must match MBASC feature names exactly (except capitalisation).
- Sparse columns removed should ameliorate misclassifications, per the README troubleshooting remedy.
- Substituting 1.4 GHz for 150 MHz radio data should degrade performance by ~4% in precision and ~3% in recall, as reported.

## Limitations

- Model performance decreases towards higher redshifts, mainly due to smaller training sample sizes.
- The raw output column does not correspond to a likelihood that a source is an AGN; histograms of it should be interpreted carefully.
- Radio data can be tricky to cross-match; wrongly cross-matched data can give bad results.
- 1.4 GHz data must be manually converted to 150 MHz by the user before use.

## Evidence

- [abstract] far-infrared and radio band importance: "we find that the far-infrared and radio bands are of great importance."
- [abstract] baseline precision/recall: "Our trained model has a precision of 0.92±0.01 and a recall of 0.87±0.02 for SFGs. For AGNs, the model performs slightly worse, with a precision of 0.87±0.02 and a recall of 0.78±0.02."
- [abstract] 1.4 GHz conversion penalty: "Converting 1.4 GHz to 150 MHz radio data reduces the performance by ~4% in precision and ~3% in recall."
- [readme] sparse column remedy: "Remove data columns with very few samples, the model might make wrong predictions based on this."
- [readme] unit requirement: "All units (except redshift) must be converted to μJy."
- [readme] header-name strictness: "it is important that the **exact** name (except for capitalisation) is used in the csv file as header. If a certain feature is not available, it can simply be left out."
