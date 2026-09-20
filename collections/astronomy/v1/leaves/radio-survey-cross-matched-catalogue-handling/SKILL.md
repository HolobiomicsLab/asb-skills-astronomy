---
name: radio-survey-cross-matched-catalogue-handling
description: Use when a flux catalogue cross-matched to a radio survey (LOFAR 150
  MHz or converted 1.4 GHz) is being prepared for MBASC classification, or when misclassifications
  suggest unit, header, cross-match, or waveband problems in the input CSV.
when_to_use_negative:
- The data are 1.4 GHz or other non-150 MHz radio fluxes that have not been converted
  — the README explicitly warns 'Ensure that you used 150 MHz radio data, not 1.4
  GHz or some other waveband.'
- The catalogue is already a fitted model output or feature-importance table rather
  than a per-source flux table needing unit/header validation.
- The goal is calibrating probability estimates — the output column 'does not have
  to correspond to a likelihood that a source is an AGN'.
edam_operation: http://edamontology.org/operation_2409
tools:
- name: MBASC
  role: Target pipeline whose exact feature-name and μJy-unit requirements define
    the catalogue validation rules; consumes the prepared CSV
  repo: github.com/Jesper-Karsten/MBASC
- name: pandas
  role: Loading and inspecting the cross-matched CSV, checking header names, unit
    consistency, and per-column sample counts
- name: lightgbm
  role: Underlying LightGBM classifier whose raw fraction output (id/output/AGN columns
    in results.csv) is interpreted after catalogue preparation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202346770
    title: A multi-band AGN-SFG classifier for extragalactic radio surveys using machine
      learning
  source_content_hash: sha256:2521221d9bdd410c58ad24cf741fde5ccc26ffdbfb9c5a917d2e8376242f19e2
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# radio-survey-cross-matched-catalogue-handling

## Summary

Preparing and validating multi-wavelength flux catalogues cross-matched to 150 MHz LOFAR radio data so they are ingestible by the MBASC/LightGBM AGN–SFG classifier: exact feature headers, μJy flux units, correct radio-band provenance, and pruning of sparse or mis-matched columns.

## When to use

Use when a flux catalogue cross-matched to a radio survey (LOFAR 150 MHz or converted 1.4 GHz) is being prepared for MBASC classification, or when misclassifications suggest unit, header, cross-match, or waveband problems in the input CSV.

## When NOT to use

- The data are 1.4 GHz or other non-150 MHz radio fluxes that have not been converted — the README explicitly warns 'Ensure that you used 150 MHz radio data, not 1.4 GHz or some other waveband.'
- The catalogue is already a fitted model output or feature-importance table rather than a per-source flux table needing unit/header validation.
- The goal is calibrating probability estimates — the output column 'does not have to correspond to a likelihood that a source is an AGN'.

## Inputs

- Multi-wavelength flux catalogue CSV (e.g. example.csv from the MBASC repo) with headers among id, redshift, u, g, r, i, z, y, J, H, K, ch1-ch4, MIPS24, PACS100, PACS160, SPIRE250, SPIRE350, SPIRE500, radiototal, radiopeak
- Flux densities in μJy for all non-redshift columns
- 150 MHz radio continuum fluxes (radiototal, radiopeak) from LOFAR Deep Field cross-matches (Lockman Hole, Boötes, ELAIS-N1)

## Outputs

- Validated input CSV accepted by MBASC with exact feature headers and μJy units
- output/results.csv with columns id, output (raw LightGBM fraction in [0,1]) and AGN (1 = predicted AGN, 0 = predicted SFG), row order matching the input file

## How to apply

Load the catalogue with pandas, ensuring every flux column (u through radiopeak) carries the exact header name from the MBASC feature table (capitalisation-insensitive but otherwise verbatim) and that all flux densities are converted to μJy, with redshift the sole unit-exempt column. Available-but-unneeded features may simply be omitted from the header rather than zero-filled, since MBASC accepts any subset of the 22 photometric/radio/redshift features. Before training, check radio provenance: the model expects 150 MHz radio flux (radiototal, radiopeak); 1.4 GHz data must be converted by the user, at a documented cost of ~4% precision and ~3% recall if converted. When predictions look wrong, drop columns with very few samples and scrutinise the radio cross-match, since wrongly cross-matched radio data can produce bad classifications. Run `python MBASC.py [path/to/file.csv]` and inspect output/results.csv (columns id, output, AGN), remembering that the raw LightGBM output is a fraction rounded to the binary AGN column and is not a likelihood.

## Related tools

- **MBASC** (Target pipeline whose exact feature-name and μJy-unit requirements define the catalogue validation rules; consumes the prepared CSV) — github.com/Jesper-Karsten/MBASC
- **pandas** (Loading and inspecting the cross-matched CSV, checking header names, unit consistency, and per-column sample counts)
- **lightgbm** (Underlying LightGBM classifier whose raw fraction output (id/output/AGN columns in results.csv) is interpreted after catalogue preparation)

## Examples

```
python MBASC.py example.csv
```

## Evaluation signals

- Every flux column header matches an MBASC feature name verbatim (e.g. ch1, MIPS24, SPIRE500, radiototal, radiopeak); any unavailable feature is simply absent, not zero-filled
- All flux values are in μJy — only redshift is unit-exempt
- radiototal/radiopeak contain 150 MHz fluxes, not unconverted 1.4 GHz values
- results.csv has exactly three columns (id, output, AGN), rows in the same order as the input file, output strictly in [0,1], and AGN equal to the rounded output
- Sparse columns and suspiciously cross-matched radio rows have been removed when misclassifications cluster on specific bands

## Limitations

- Wrongly cross-matched radio data can give bad results, and radio data are flagged as 'tricky to cross-match' in the README troubleshooting guidance
- Using unconverted 1.4 GHz data instead of 150 MHz reduces performance by ~4% in precision and ~3% in recall per the paper
- The raw LightGBM output is a fraction that depends on dataset-wide classifier quality, not a per-source AGN probability; histograms should be interpreted cautiously
- Model performance decreases towards higher redshifts, mainly due to smaller training sample sizes

## Evidence

- [readme] Unit and header requirements: "it is important that the **exact** name (except for capitalisation) is used in the csv file as header. If a certain feature is not available, it can simply be left out. All units (except redshift)"
- [readme] Radio cross-match failure mode: "Play around with removing some features from the data, particurarly radio data can be tricky to cross-match. Wrongly cross-matched data can give bad results."
- [readme] Waveband provenance check: "Ensure that you used 150 MHz radio data, not 1.4 GHz or some other waveband."
- [readme] Output schema interpretation: "The results.csv file has 3 columns: id, output, and AGN. The ID column is the given ID, or if not given a simple numeric value. The results.csv is in the same order as the input file."
- [abstract] 1.4 GHz conversion cost: "Converting 1.4 GHz to 150 MHz radio data reduces the performance by ~4% in precision and ~3% in recall."
