---
name: mad-flux-outlier-flagging
description: Use when building S-BART stellar templates from reduced S2D HARPS/ESPRESSO
  observations and individual spectral frames contain anomalous pixel fluxes (cosmic
  rays, detector artifacts) that must be excluded before co-adding orders into the
  template.
when_to_use_negative:
- The goal is to sigma-clip whole-exposure RVs against the DRS estimate rather than
  to flag per-pixel flux outliers within frames
- Frames already pass order-wise SNR checks and warning-keyword flag rejection and
  contain no pixel-level flux anomalies, so MAD flagging adds no protection
tools:
- name: S-BART / SBART
  role: Implements the MAD-based flux-outlier flagging as a configurable frame-selection
    option during DataClass loading, prior to order-by-order stellar template creation.
  repo: https://github.com/iastro-pt/sBART
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202142262
    title: A novel framework for semi-Bayesian radial velocities through template
      matching
  source_content_hash: sha256:fcc63af4be589d778661c99f473b20c39506c0ade327f1192299bdc79e698fd0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/iastro-pt/sBART
  tools:
  - S-BART
  - S-BART / SBART
  - SBART
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# mad-flux-outlier-flagging

## Summary

An optional frame-preparation step in S-BART that flags flux-outlier pixels in reduced S2D spectral frames using a MAD-based criterion before the order-by-order stellar template is co-added, protecting the template from corrupted pixels that would bias the semi-Bayesian RV fit.

## When to use

Use when building S-BART stellar templates from reduced S2D HARPS/ESPRESSO observations and individual spectral frames contain anomalous pixel fluxes (cosmic rays, detector artifacts) that must be excluded before co-adding orders into the template.

## When NOT to use

- The goal is to sigma-clip whole-exposure RVs against the DRS estimate rather than to flag per-pixel flux outliers within frames
- Frames already pass order-wise SNR checks and warning-keyword flag rejection and contain no pixel-level flux anomalies, so MAD flagging adds no protection

## Inputs

- Reduced S2D spectral observation frames (HARPS or ESPRESSO), loaded into the SBART DataClass
- SBART frame-selection configuration enabling the MAD flux-outlier flagging option, optionally together with order-wise SNR checks and warning-keyword flag rejection

## Outputs

- Flux-outlier pixel flags/masks per frame and spectral order, applied during stellar-template creation
- Stellar template written to disk as FITS files per sub-Instrument, with text files summarizing included and rejected data
- Metrics plots of the number of pixels rejected in each spectral order

## How to apply

Enable the option 'Allow to use MAD to flag flux outliers' in the SBART configuration while loading the reduced S2D observations into the SBART DataClass. The MAD-based criterion is applied per spectral order to identify pixel fluxes that deviate strongly from the local median absolute deviation, marking them as flux outliers so they are excluded from the template-creation routine. This flagging runs alongside the other frame-selection conditions — order-wise SNR checks and rejection of frames flagged by warning keyword flags — and before combining selected observations order-by-order into a single stellar template per sub-Instrument. After template construction, generate the metrics plots reporting the amount of pixels rejected in each spectral order: if rejections cluster chromatically (concentrated in specific orders), this diagnoses chromatic issues or outlier observations rather than isolated bad pixels.

## Related tools

- **S-BART / SBART** (Implements the MAD-based flux-outlier flagging as a configurable frame-selection option during DataClass loading, prior to order-by-order stellar template creation.) — https://github.com/iastro-pt/sBART

## Evaluation signals

- Stellar-template FITS file exists in the templates folder and is loadable by SBART
- Per-order pixel-rejection metric plots were produced for each sub-Instrument
- Pixel rejections are distributed as isolated outliers; rejections concentrated in specific spectral orders indicate chromatic issues or an outlier observation that should be re-inspected
- Text files accompanying the template list which observations were included versus rejected

## Limitations

- The MAD criterion is an optional configuration ('Allow to use MAD to flag flux outliers'); leaving it disabled leaves template co-adding exposed to flux-outlier pixels.
- High per-order rejection counts do not automatically mean correct filtering — they can signal chromatic issues or an outlier observation requiring manual review.
- Template quality directly affects RV precision: the authors attribute performance differences between pipelines to differences in stellar-template construction and telluric handling.

## Evidence

- [methods] MAD flagging is an explicit SBART option: "Allow to use MAD to flag flux outliers."
- [abstract] Template built from available observations feeds the per-order common-RV-shift model: "a common RV shift is assumed to describe the difference between each spectral order of a given stellar spectrum and a template built from the available observations."
- [methods] Distinct, alternative RV-level sigma-clipping filter step: "Sigma-clipping RVs based on the DRS estimate"
