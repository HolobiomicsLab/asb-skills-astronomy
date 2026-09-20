---
name: primary-beam-correction
description: Use when a chimenea-style pipeline has produced cleaned per-epoch CASA
  images (from masked and open Cleans seeded by the deep-image source mask) and the
  epoch images must be primary-beam corrected and exported as FITS files.
when_to_use_negative:
- Input epoch images are already primary-beam-corrected FITS products — reapplying
  the correction would distort the flux scale
- Data are raw, uncalibrated AMI-LA visibility files — chimenea and this stage assume
  pre-calibrated measurement sets; use the drive-ami/AMI-REDUCE calibration path first
- Fields dominated by extended emission or complex blended sources — the paper states
  such fields still need manual reduction, and the automated per-epoch flow this correction
  closes off is not appropriate
tools:
- name: chimenea
  role: Implements the heuristic imaging algorithm whose final stage applies primary-beam
    correction to each epoch image and exports FITS; key logic-flow lives in chimenea/pipeline.py
  repo: https://github.com/timstaley/chimenea
- name: CASA
  role: Provides the underlying image-synthesis subroutines, including the primary-beam
    correction applied to the per-epoch images
  repo: http://casa.nrao.edu
- name: drive-casa
  role: Python interface layer through which chimenea invokes the CASA subroutines
    (Clean, PB correction) from a generic Python process
  repo: https://github.com/timstaley/drive-casa
- name: AMIsurvey
  role: End-to-end AMI-LA pipeline that ties drive-ami, drive-casa and chimenea together,
    producing calibrated, cleaned, PB-corrected multi-epoch images
  repo: https://github.com/timstaley/amisurvey
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.08.003
    title: 'Chimenea and other tools: Automated imaging of multi-epoch radio-synthesis
      data with CASA'
  source_content_hash: sha256:2a1c1782bd7190193b0e675d8ef2315f414ae71ce311403d882fcac823c57701
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/timstaley/chimenea
  tools:
  - AMIsurvey
  - CASA
  - chimenea
  - drive-casa
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# Primary-beam correction of per-epoch radio-synthesis images

## Summary

Final-stage correction in the chimenea multi-epoch imaging dataflow: after the deep image, masked per-epoch and open per-epoch Cleans are complete, each epoch image is primary-beam corrected via CASA subroutines and exported as FITS, so the per-epoch products are flux-scaled and ready for source extraction and transient identification.

## When to use

Use when a chimenea-style pipeline has produced cleaned per-epoch CASA images (from masked and open Cleans seeded by the deep-image source mask) and the epoch images must be primary-beam corrected and exported as FITS files.

## When NOT to use

- Input epoch images are already primary-beam-corrected FITS products — reapplying the correction would distort the flux scale
- Data are raw, uncalibrated AMI-LA visibility files — chimenea and this stage assume pre-calibrated measurement sets; use the drive-ami/AMI-REDUCE calibration path first
- Fields dominated by extended emission or complex blended sources — the paper states such fields still need manual reduction, and the automated per-epoch flow this correction closes off is not appropriate

## Inputs

- Per-epoch cleaned CASA images from the masked and open per-epoch Clean stages
- Pre-calibrated multi-epoch radio-synthesis measurement-set files
- Clean mask image and source model derived from the concatenated-epoch deep image (upstream products that constrain the per-epoch images being corrected)

## Outputs

- Primary-beam-corrected image per epoch
- FITS file for each corrected epoch image, ready for source extraction and transient identification

## How to apply

Run this as the last step of the chimenea imaging flow, after the concatenated-epoch deep image has been made with iterative RMS re-estimation during repeated Clean, a Clean mask has been built from automated source detection on the deep image, and each epoch has undergone a masked Clean followed by an open Clean initialized from the masked-epoch source model. Take the per-epoch cleaned CASA images (the open per-epoch Clean products, which recover any emission outside the deep-image mask) and drive the CASA primary-beam correction subroutine through the drive-casa interface, so correction runs from the same generic Python process that drives Clean. Export each corrected epoch image as a FITS file; the paper's acceptance criterion is that the pipeline module runs end-to-end on the (simulated) pre-calibrated multi-epoch measurement sets and that every epoch yields a primary-beam-corrected FITS image.

## Related tools

- **chimenea** (Implements the heuristic imaging algorithm whose final stage applies primary-beam correction to each epoch image and exports FITS; key logic-flow lives in chimenea/pipeline.py) — https://github.com/timstaley/chimenea
- **CASA** (Provides the underlying image-synthesis subroutines, including the primary-beam correction applied to the per-epoch images) — http://casa.nrao.edu
- **drive-casa** (Python interface layer through which chimenea invokes the CASA subroutines (Clean, PB correction) from a generic Python process) — https://github.com/timstaley/drive-casa
- **AMIsurvey** (End-to-end AMI-LA pipeline that ties drive-ami, drive-casa and chimenea together, producing calibrated, cleaned, PB-corrected multi-epoch images) — https://github.com/timstaley/amisurvey

## Evaluation signals

- The pipeline module runs end-to-end on the simulated pre-calibrated multi-epoch measurement sets without manual intervention
- Every epoch yields a primary-beam-corrected FITS image — the paper's stated validation criterion
- Each corrected epoch image corresponds one-to-one with an input epoch measurement set (no epochs dropped or merged)
- Corrected epoch images retain the deep-image source positions where Clean model components were restricted to the vicinity of known sources
- Downstream flux-density measurements on corrected FITS images are consistent with the iterative RMS-estimation / Clean-bias-avoidance design (no negative bowl or suppressed fluxes toward the field centre)

## Limitations

- Fields containing extended emission or complex blended sources still need to be manually reduced
- Reduction time was benchmarked only on small ALARRM datasets, with file sizes on the order of a gigabyte or less; for a large dataset the repeated re-Cleaning may become a significant contribution to overall processing time
- chimenea has only been tested with a limited set of telescope configurations (telescope-agnostic by design but not exhaustively validated)

## Evidence

- [full_text] Apply primary-beam correction to each epoch image and export the corrected images as FITS files.: [evidence span withheld — anchor status: not_found]
- [intro] AMIsurvey produces calibrated and cleaned multi-epoch images: [evidence span withheld — anchor status: not_found]
