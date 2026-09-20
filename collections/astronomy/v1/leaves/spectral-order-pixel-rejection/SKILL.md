---
name: spectral-order-pixel-rejection
description: Use when building a stellar template from reduced S2D echelle observations
  and you must decide which frames and which pixels per spectral order are trustworthy
  enough to enter the co-add, prior to RV estimation.
when_to_use_negative:
- When RVs are computed by per-order weighted-average template matching (the classical
  variant) rather than S-BART's common-shift semi-Bayesian framework, since the per-order
  rejection context differs
- When the input is 1D-merged or CCF-based data rather than S2D echelle orders, since
  the MAD flagging and per-order rejection operate order-wise on S2D flux
- When analyzing planetary or activity signals in the resulting RV series — the article
  defers such full analysis to future work; this skill only covers data/template quality
  control
tools:
- name: S-BART
  role: Pipeline whose stellar-template creation routine performs MAD-based flux-outlier
    flagging, order-wise SNR frame selection, and per-order pixel-rejection metrics
    during co-addition
  repo: https://github.com/iastro-pt/sBART
- name: HARPS
  role: Instrument supplying the reduced S2D spectral orders that the pixel-rejection
    filters operate on
- name: ESPRESSO
  role: Instrument supplying S2D observations (33 GTO targets) to which the same template-construction
    and rejection routine is applied
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1051/0004-6361/202142262
    title: A novel framework for semi-Bayesian radial velocities through template
      matching
  source_content_hash: sha256:62692c5e75cc89ba9bac0fe50f85fb26c8fe6662653b5260dbfb72424d922b4a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iastro-pt/sBART
  tools:
  - ESPRESSO
  - HARPS
  - S-BART
  - SBART
  - SERVAL
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# spectral-order-pixel-rejection

## Summary

S-BART's MAD-based flagging of flux-outlier pixels combined with order-wise SNR checks and warning-keyword frame rejection, applied before building the per-sub-Instrument stellar template from HARPS/ESPRESSO S2D spectral orders.

## When to use

Use when building a stellar template from reduced S2D echelle observations and you must decide which frames and which pixels per spectral order are trustworthy enough to enter the co-add, prior to RV estimation.

## When NOT to use

- When RVs are computed by per-order weighted-average template matching (the classical variant) rather than S-BART's common-shift semi-Bayesian framework, since the per-order rejection context differs
- When the input is 1D-merged or CCF-based data rather than S2D echelle orders, since the MAD flagging and per-order rejection operate order-wise on S2D flux
- When analyzing planetary or activity signals in the resulting RV series — the article defers such full analysis to future work; this skill only covers data/template quality control

## Inputs

- Reduced HARPS or ESPRESSO S2D spectral frames (echelle orders)
- Frame metadata/keyword flags (warning KW flags, order-wise SNR values)
- Existing stellar template FITS files when extending a template

## Outputs

- Per-sub-Instrument stellar template stored as FITS files in the templates folder
- Text files summarizing included and rejected frames/data
- Metrics plots of the amount of pixels rejected in each spectral order
- Flux-outlier pixel masks (MAD-flagged) consumed by template construction

## How to apply

Load the reduced S2D frames into the SBART DataClass and apply frame-selection conditions: an order-wise SNR check and rejection of frames flagged by warning keyword flags. Enable the 'Allow to use MAD to flag flux outliers' option so that pixels deviating per the MAD criterion are flagged within each spectral order before the order-by-order co-add. Combine the surviving observations order-by-order into a single stellar template per sub-Instrument, write the template as FITS files in the templates folder alongside text files summarizing included and rejected data, and generate metrics plots of the number of pixels rejected per spectral order. Chromatic rejection patterns in those plots diagnose telluric or instrumental issues; the rejected-pixel handling matters because S-BART assumes a single common RV shift describes the difference between each spectral order and the template, so contaminated orders directly bias the semi-Bayesian RV posterior.

## Related tools

- **S-BART** (Pipeline whose stellar-template creation routine performs MAD-based flux-outlier flagging, order-wise SNR frame selection, and per-order pixel-rejection metrics during co-addition) — https://github.com/iastro-pt/sBART
- **HARPS** (Instrument supplying the reduced S2D spectral orders that the pixel-rejection filters operate on)
- **ESPRESSO** (Instrument supplying S2D observations (33 GTO targets) to which the same template-construction and rejection routine is applied)

## Evaluation signals

- The stellar template FITS file exists in the templates folder and is loadable by SBART
- Per-order pixel-rejection metrics plots were produced for each sub-Instrument
- No single spectral order shows an anomalously small RV uncertainty in the RV_orderwise_errors.png diagnostic (would indicate a contaminated order skewing the fit)
- The text summaries of included vs. rejected frames match the frame-selection conditions actually applied (SNR checks, warning KW flags)
- Downstream check: standard deviation of S-BART RVs on Barnard's star HARPS data is smaller than HARPS-TERRA and SERVAL, consistent with clean template construction

## Limitations

- Aggressive rejection can remove chromatically informative pixels; the article attributes performance differences between pipelines partly to template construction and telluric handling
- Telluric-contaminated pixels are a known failure mode — telluric handling (Telfit/GDAS-based) interacts with which pixels survive rejection
- Telfit fallback exists for reference frames where GDAS profiles cannot be downloaded, affecting which frames contribute to telluric correction before template construction

## Evidence

- [methods] Allow to use MAD to flag flux outliers.: "Allow to use MAD to flag flux outliers."
- [methods] check order-wise SNR: "Added a new condition to check order-wise SNR"
- [abstract] common RV shift describes template difference: "a common RV shift is assumed to describe the difference between each spectral order of a given stellar spectrum and a template built from the available observations."
- [abstract] performance differences attributed to template construction: "We believe this is due to differences in the construction of the stellar template and the handling of telluric features."
