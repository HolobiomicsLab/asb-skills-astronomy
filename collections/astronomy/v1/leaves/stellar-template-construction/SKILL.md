---
name: stellar-template-construction
description: Use when reduced S2D spectra (HARPS or ESPRESSO) must be combined order-by-order
  into a stellar template to serve as the reference for template-matching RV extraction
  or to study template-quality effects on RV precision.
when_to_use_negative:
- The goal is per-order RV estimation combined by weighted average (traditional template
  matching) rather than building a co-added template for the semi-Bayesian common-RV-shift
  model
- Telluric templates are the target product — that requires TelFit with GDAS atmospheric
  profiles, not the stellar-template routine
- Only a single observation exists per target, so co-adding frames into a template
  adds no information
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: SBART
  role: 'Executes the stellar-template creation routine: frame selection, order-by-order
    co-addition, FITS template output, and rejection metric plots'
  repo: https://github.com/iastro-pt/sBART
- name: Telfit
  role: Separately installed companion used to build the telluric template from the
    same observations (with GDAS profiles); distinct from the stellar-template step
  repo: https://github.com/kgullikson88/Telluric-Fitter
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202142262
    title: A novel framework for semi-Bayesian radial velocities through template
      matching
  source_content_hash: sha256:0bc38746206a5501589299ec25d2cea445d0d2cb922152cc0dd7ae622f9da858
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iastro-pt/sBART
  tools:
  - ESO official pipeline
  - S-BART
  - SBART
  - SERVAL
  - Telfit
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# stellar-template-construction

## Summary

Build a high-S/N stellar template by co-adding the spectral orders of all quality-selected observations per sub-Instrument with S-BART; the template is the reference against which the semi-Bayesian RV shift of each spectrum is fitted and is credited for S-BART's smaller RV scatter than HARPS-TERRA and SERVAL.

## When to use

Use when reduced S2D spectra (HARPS or ESPRESSO) must be combined order-by-order into a stellar template to serve as the reference for template-matching RV extraction or to study template-quality effects on RV precision.

## When NOT to use

- The goal is per-order RV estimation combined by weighted average (traditional template matching) rather than building a co-added template for the semi-Bayesian common-RV-shift model
- Telluric templates are the target product — that requires TelFit with GDAS atmospheric profiles, not the stellar-template routine
- Only a single observation exists per target, so co-adding frames into a template adds no information

## Inputs

- Reduced S2D spectral frames (HARPS archival or ESPRESSO GTO) for a target
- Frame-selection configuration (order-wise SNR check, warning keyword flags, MAD flux-outlier flagging, NORMALIZATION_MODE)

## Outputs

- Stellar template per sub-Instrument stored as FITS files in the templates folder
- Text files summarizing included and rejected observations
- Per-order pixel-rejection metric plots for each sub-Instrument

## How to apply

Load the reduced S2D frames into the SBART DataClass and apply frame selection: order-wise SNR checks, rejection of frames flagged by warning keyword flags, and MAD-based flagging of flux-outlier pixels (the 'Allow to use MAD to flag flux outliers' option). Combine the surviving observations order-by-order into one stellar template per sub-Instrument using SBART's stellar-template creation routine, optionally selecting a NORMALIZATION_MODE for bulk-normalizing the stellar spectra. Write the template to disk as FITS files in the templates folder alongside text files summarizing included and rejected frames. Generate metric plots of the number of pixels rejected in each spectral order to diagnose chromatic issues or outlier observations before launching the RV step; template construction is assumed to underpin the common-RV-shift model (one shift per spectrum across all orders) in the semi-Bayesian framework.

## Related tools

- **SBART** (Executes the stellar-template creation routine: frame selection, order-by-order co-addition, FITS template output, and rejection metric plots) — https://github.com/iastro-pt/sBART
- **Telfit** (Separately installed companion used to build the telluric template from the same observations (with GDAS profiles); distinct from the stellar-template step) — https://github.com/kgullikson88/Telluric-Fitter

## Examples

```
pip install SBART; sbart load S2D frames -> stellar template creation routine -> templates/<subInstrument>/template FITS
```

## Evaluation signals

- The stellar template FITS file exists in the templates folder and is loadable by SBART
- Per-order pixel-rejection metric plots were produced for each sub-Instrument
- Text files of included and rejected data were written alongside the template
- Frames entering the template passed the order-wise SNR check and were not rejected by warning keyword flags or MAD flux-outlier flagging
- Downstream RVs fitted against this template show lower standard deviation than HARPS-TERRA and SERVAL on the same data (e.g., Barnard's star HARPS archival series)

## Limitations

- Pathological observations or chromatic issues surface as anomalous per-order pixel rejection and require inspection of the metric plots before RV extraction
- Template construction (together with telluric handling) is the authors' attributed cause of performance differences versus other pipelines, so poor frame selection directly degrades RV precision
- pysynphot or Telfit dependency failures can break the companion telluric-template workflow that uses the same selected frames

## Evidence

- [abstract] S-BART builds a stellar template from the available observations and assumes a single common RV shift describes the difference between each spectral order of a given stellar spectrum and that template: "a common RV shift is assumed to describe the difference between each spectral order of a given stellar spectrum and a template built from the available observations."
- [methods] Frame selection combines order-wise SNR checks, warning keyword flags, and MAD-based flux-outlier flagging before template construction: "Allow to use MAD to flag flux outliers."
- [methods] Normalization mode controls how stellar spectra are normalized for template construction: "The "NORMALIZATION_MODE" will be used to select the method that will be used to normalize the stellar spectra"
- [readme] SBART is pip-installable, per the project documentation: "s-BART: A semi-Bayesian implementation of template matching for precise Radial Velocities"
