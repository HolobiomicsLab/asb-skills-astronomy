---
name: multi-epoch-deep-image-stacking
description: Use when you hold pre-calibrated multi-epoch radio-synthesis measurement
  sets and need steady-source detection plus per-epoch cleaned images, e.g. ALARRM-style
  transient follow-up with AMI-LA or any telescope via CASA.
when_to_use_negative:
- Input data is raw (uncalibrated) AMI-LA datafiles — chimenea assumes pre-calibrated
  measurement sets; calibration belongs to drive-ami/AMI-REDUCE upstream
- Fields dominated by extended emission or complex blended sources — the paper notes
  such fields 'still need to be manually reduced'
- Single-epoch data with no multi-epoch counterpart, where a deep-image-derived mask
  provides no steady-source leverage
edam_operation: http://edamontology.org/operation_3443
tools:
- name: chimenea
  role: Implements the telescope-agnostic deep-image + masked/open per-epoch Clean
    algorithm; key logic-flow in chimenea/pipeline.py
  repo: https://github.com/timstaley/chimenea
- name: drive-casa
  role: Python interface layer used to invoke CASA Clean subroutines for concatenation,
    masked Clean, and open Clean
  repo: https://github.com/timstaley/drive-casa
- name: CASA
  role: Provides the underlying image-synthesis (Clean) subroutines for deep stacking
    and per-epoch deconvolution
- name: AMIsurvey
  role: Telescope-specific end-to-end AMI-LA pipeline that wraps chimenea to produce
    calibrated and cleaned multi-epoch images
  repo: https://github.com/timstaley/amisurvey
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.08.003
    title: 'Chimenea and other tools: Automated imaging of multi-epoch radio-synthesis
      data with CASA'
  source_content_hash: sha256:0a622faa8aab039c4d52e8cd7ae58b0d324031e2d9fd906d6902950df37c90f2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/timstaley/chimenea
  tools:
  - AMIsurvey
  - CASA
  - chimenea
  - drive-casa
  edam_operation: http://edamontology.org/operation_3443
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# multi-epoch-deep-image-stacking

## Summary

Chimenea-style stacking of pre-calibrated multi-epoch radio-synthesis measurement sets into a single concatenated 'deep' image via iterated CASA Clean, from which a Clean mask is built to constrain masked and open per-epoch imaging.

## When to use

Use when you hold pre-calibrated multi-epoch radio-synthesis measurement sets and need steady-source detection plus per-epoch cleaned images, e.g. ALARRM-style transient follow-up with AMI-LA or any telescope via CASA.

## When NOT to use

- Input data is raw (uncalibrated) AMI-LA datafiles — chimenea assumes pre-calibrated measurement sets; calibration belongs to drive-ami/AMI-REDUCE upstream
- Fields dominated by extended emission or complex blended sources — the paper notes such fields 'still need to be manually reduced'
- Single-epoch data with no multi-epoch counterpart, where a deep-image-derived mask provides no steady-source leverage

## Inputs

- Pre-calibrated multi-epoch radio-synthesis measurement-set files (telescope-agnostic)
- Concatenated deep CASA image covering all epochs
- Automated source-detection catalogue from the deep image

## Outputs

- Deep concatenated Clean image with iteratively re-estimated background RMS
- Clean mask image built from deep-image source components
- Per-epoch masked-Clean images and subsequent open-Clean images
- Primary-beam-corrected FITS images, one per epoch

## How to apply

First concatenate all epochs into a single deep image using the CASA Clean task driven through drive-casa, re-running Clean iteratively while the background RMS is re-estimated at each pass to avoid Clean-bias. Next run automated source-detection on the deep image and build a Clean mask image from the detected source components, since the deep image isolates steady sources across epochs. Then apply a masked Clean to each individual epoch, restricting deconvolution to the deep-image-derived mask, followed by an open (unmasked) Clean per epoch initialized from the masked-epoch source model so emission outside the mask can be recovered. Finally apply primary-beam correction to each epoch image and export as FITS. The deep-image-first rationale is that Clean model components are placed only in the vicinity of known sources, guarding against noise-driven components in low-SNR single epochs.

## Related tools

- **chimenea** (Implements the telescope-agnostic deep-image + masked/open per-epoch Clean algorithm; key logic-flow in chimenea/pipeline.py) — https://github.com/timstaley/chimenea
- **drive-casa** (Python interface layer used to invoke CASA Clean subroutines for concatenation, masked Clean, and open Clean) — https://github.com/timstaley/drive-casa
- **CASA** (Provides the underlying image-synthesis (Clean) subroutines for deep stacking and per-epoch deconvolution)
- **AMIsurvey** (Telescope-specific end-to-end AMI-LA pipeline that wraps chimenea to produce calibrated and cleaned multi-epoch images) — https://github.com/timstaley/amisurvey

## Examples

```
from chimenea import pipeline  # key logic-flow in chimenea/pipeline.py, driving CASA Clean via drive-casa
```

## Evaluation signals

- Each epoch yields a primary-beam-corrected FITS image after the pipeline module runs end-to-end
- Background RMS is re-estimated at every Clean pass (no single fixed threshold carried through the stack)
- Clean mask image exists and its components coincide with sources auto-detected on the concatenated deep image
- In masked per-epoch Cleans, Clean model components are placed only in the vicinity of known (deep-image) sources
- The open per-epoch Clean, initialized from the masked-epoch source model, recovers emission outside the mask without diverging

## Limitations

- Fields containing extended emission or complex blended sources still require manual reduction
- Iterative re-Cleaning can become a significant fraction of total processing time for large datasets
- Benchmarked only on small ALARRM datasets (order of a gigabyte or less, reduction times on the order of minutes)

## Evidence

- [full_text] deep-image-derived mask constrains per-epoch Clean: [evidence span withheld — anchor status: not_found]
- [discussion] repeated Clean with RMS re-estimation: [evidence span withheld — anchor status: not_found]
