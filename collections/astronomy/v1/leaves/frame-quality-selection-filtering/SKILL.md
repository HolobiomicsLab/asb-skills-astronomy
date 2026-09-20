---
name: frame-quality-selection-filtering
description: Use when preparing reduced S2D Echelle observations (HARPS, ESPRESSO)
  for SBART template creation or RV extraction, and you must decide which frames and
  pixels are trustworthy before co-adding spectra into a stellar template.
when_to_use_negative:
- Frames or spectra already validated by another pipeline's quality-control stage
  and supplied as a pre-screened list — re-applying SNR/KW rejection may discard usable
  data
- Analysis of already-co-added or template-matched products (e.g., existing template
  FITS files); this skill operates on raw frame selection before template construction
- Spectral orders affected by telluric residuals where the correct action is telluric
  handling (e.g., Telfit-based modelling) rather than frame rejection
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: SBART
  role: Implements the frame-selection conditions (order-wise SNR check, warning-KW
    rejection, MAD flux-outlier flagging) within its DataClass before template creation
  repo: https://github.com/iastro-pt/sBART
- name: HARPS
  role: Instrument whose archival S2D observations (e.g., Barnard's star) are filtered
    and validated with this skill
- name: ESPRESSO
  role: Instrument whose S2D GTO observations are frame-selected before RV extraction
    and template building
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202142262
    title: A novel framework for semi-Bayesian radial velocities through template
      matching
  source_content_hash: sha256:36c0ef3117f2b7b00ee0360ea770c0be48ed1f1befa046ffcd3eb72c92b4788b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iastro-pt/sBART
  tools:
  - ESPRESSO
  - HARPS
  - S-BART
  - SBART
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# frame-quality-selection-filtering

## Summary

Selects which reduced S2D spectroscopic frames enter SBART's stellar-template construction and RV computation, using order-wise SNR checks, warning-keyword flag rejection, MAD-based flux-outlier pixel flagging, and sigma-clipping of RVs against the DRS estimate. Quality filtering upstream of template matching is essential because contaminated frames distort the co-added template that S-BART's semi-Bayesian RV estimation relies on.

## When to use

Use when preparing reduced S2D Echelle observations (HARPS, ESPRESSO) for SBART template creation or RV extraction, and you must decide which frames and pixels are trustworthy before co-adding spectra into a stellar template.

## When NOT to use

- Frames or spectra already validated by another pipeline's quality-control stage and supplied as a pre-screened list — re-applying SNR/KW rejection may discard usable data
- Analysis of already-co-added or template-matched products (e.g., existing template FITS files); this skill operates on raw frame selection before template construction
- Spectral orders affected by telluric residuals where the correct action is telluric handling (e.g., Telfit-based modelling) rather than frame rejection

## Inputs

- Reduced S2D Echelle spectra (HARPS or ESPRESSO observations) loaded into the SBART DataClass
- Frame metadata including order-wise SNR values and warning keyword (KW) flags
- DRS RV estimates used as reference for sigma-clipping
- Telluric/GDAS-related keywords (e.g., water-related keywords) associated with each frame

## Outputs

- A filtered set of frames per sub-Instrument entering stellar-template creation
- Per-order maps of rejected/outlier pixels (MAD-flagged)
- Metric plots showing the amount of pixels rejected in each spectral order
- Text files summarizing included and rejected data, written alongside the template FITS files

## How to apply

Load the reduced S2D observations into the SBART DataClass and apply frame-selection conditions: check order-wise SNR and reject frames flagged by warning keyword flags, with user-controlled thresholds for finer frame selection. Enable the 'Allow to use MAD to flag flux outliers' option so outlier pixels within accepted frames are flagged by a MAD-based criterion before template co-adding. Additionally, sigma-clip RVs based on the DRS estimate to discard discrepant measurements, and select the spectral normalization method via the NORMALIZATION_MODE keyword. Diagnose the filtering by inspecting metric plots of the amount of pixels rejected in each spectral order, which reveal chromatic issues or outlier observations.

## Related tools

- **SBART** (Implements the frame-selection conditions (order-wise SNR check, warning-KW rejection, MAD flux-outlier flagging) within its DataClass before template creation) — https://github.com/iastro-pt/sBART
- **HARPS** (Instrument whose archival S2D observations (e.g., Barnard's star) are filtered and validated with this skill)
- **ESPRESSO** (Instrument whose S2D GTO observations are frame-selected before RV extraction and template building)

## Examples

```
pip install SBART; from SBART.Instruments import HARPS; harps = HARPS(frame_path='S2D/', frame_usage_mode='all', user_configs={'reject_frames_KW': True, 'allow_MAD_flagging': True})
```

## Evaluation signals

- The stellar template FITS file exists in the templates folder and is loadable by SBART for each sub-Instrument
- Per-order pixel-rejection metric plots were produced for each sub-Instrument and show plausible rejection rates (no order with near-total rejection, which would indicate chromatic issues or a bad observation)
- Text summary files correctly list which frames were included and which were rejected along with the triggering condition (SNR, KW flag)
- Sigma-clipped RV set does not contain measurements far from the DRS RV estimate used as the clipping reference
- RV scatter of the filtered dataset is consistent with (i.e., not worse than) published S-BART results, e.g., smaller scatter than HARPS-TERRA and SERVAL on Barnard's star HARPS data

## Limitations

- MAD-based pixel flagging and warning-KW rejection thresholds may differ across sub-Instruments; a single global setting can over- or under-reject for one instrument
- Telluric template creation can fail without water-related keywords, and GDAS profiles may be unavailable for some reference frames, complicating frame quality assessment
- Sigma-clipping against DRS RVs assumes the DRS estimate is itself reliable; a corrupted DRS value can propagate bad clipping decisions

## Evidence

- [methods] Load the reduced S2D observations into the SBART DataClass, applying frame-selection conditions including order-wise SNR checks and rejection of frames flagged by warning keyword flags.: [evidence span withheld — anchor status: not_found]
- [methods] MAD filter step: "Allow to use MAD to flag flux outliers."
- [methods] Sigma-clipping filter step: "Sigma-clipping RVs based on the DRS estimate"
- [methods] Template validation and diagnostics from source card workflow: [evidence span withheld — anchor status: not_found]
- [abstract] Motivation for careful frame and template handling: "We believe this is due to differences in the construction of the stellar template and the handling of telluric features."
