---
name: fits-image-export
description: Use when a per-epoch radio-synthesis image produced by CASA Clean must
  be made telescope-portable and consumable by external tools, i.e. after deconvolution,
  requiring primary-beam correction and FITS export.
when_to_use_negative:
- Input data are raw AMI-LA files still requiring calibration — the AMI-REDUCE calibration
  via drive-ami must run first; this skill only handles the post-Clean imaging export
  stage
- Fields containing extended emission or complex blended sources that need manual
  reduction, since the automated pipeline does not handle such sources
- Non-image-synthesis products (e.g. UV visibility tables or calibration tables) that
  are not epoch images to be corrected and exported
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3321
tools:
- name: chimenea
  role: Implements the imaging algorithm whose final step applies primary-beam correction
    and exports each epoch image as FITS; key logic-flow in chimenea/pipeline.py
  repo: https://github.com/timstaley/chimenea
- name: drive-casa
  role: Python interface layer used to invoke the CASA imaging subroutines that perform
    the Clean and primary-beam correction/export steps
  repo: https://github.com/timstaley/drive-casa
- name: CASA
  role: Provides the underlying image-synthesis operations, including primary-beam
    correction and FITS export of cleaned epoch images
- name: AMIsurvey
  role: End-to-end telescope-specific pipeline that ties the stages together, producing
    calibrated and cleaned multi-epoch images ready for source extraction and transient
    identification
  repo: https://github.com/timstaley/amisurvey
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.08.003
    title: 'Chimenea and other tools: Automated imaging of multi-epoch radio-synthesis
      data with CASA'
  source_content_hash: sha256:cf3f9e18841bb74326b7e2ca1c8d5b78aa3dba1452cf33c68c0035cf50c021de
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/timstaley/chimenea
  tools:
  - AMIsurvey
  - CASA
  - chimenea
  - drive-casa
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3321
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# fits-image-export

## Summary

Final stage of the chimenea multi-epoch imaging dataflow: apply primary-beam correction to each per-epoch Clean image and export the corrected images as FITS files for downstream source extraction and transient identification.

## When to use

Use when a per-epoch radio-synthesis image produced by CASA Clean must be made telescope-portable and consumable by external tools, i.e. after deconvolution, requiring primary-beam correction and FITS export.

## When NOT to use

- Input data are raw AMI-LA files still requiring calibration — the AMI-REDUCE calibration via drive-ami must run first; this skill only handles the post-Clean imaging export stage
- Fields containing extended emission or complex blended sources that need manual reduction, since the automated pipeline does not handle such sources
- Non-image-synthesis products (e.g. UV visibility tables or calibration tables) that are not epoch images to be corrected and exported

## Inputs

- Per-epoch cleaned CASA images (from masked and open Clean runs) and their Clean models
- Pre-calibrated multi-epoch radio-synthesis measurement-set files
- Clean mask derived from the concatenated deep image

## Outputs

- Primary-beam-corrected FITS image files, one per epoch
- Per-epoch images suitable for source extraction and transient identification

## How to apply

After the masked and open per-epoch Cleans have produced cleaned images and Clean models for each epoch, apply a primary-beam correction to each epoch image so fluxes are correctable across the field of view. Then export each primary-beam-corrected image to FITS format, one file per epoch, so that images are ready for source extraction and transient identification as in the AMIsurvey/ALARRM end-to-end pipeline. Corrected images are produced via CASA imaging subroutines driven through drive-casa from the chimenea pipeline.py module. The stage is validated by confirming the pipeline module runs end-to-end on the simulated measurement sets and that each epoch yields a primary-beam-corrected FITS image.

## Related tools

- **chimenea** (Implements the imaging algorithm whose final step applies primary-beam correction and exports each epoch image as FITS; key logic-flow in chimenea/pipeline.py) — https://github.com/timstaley/chimenea
- **drive-casa** (Python interface layer used to invoke the CASA imaging subroutines that perform the Clean and primary-beam correction/export steps) — https://github.com/timstaley/drive-casa
- **CASA** (Provides the underlying image-synthesis operations, including primary-beam correction and FITS export of cleaned epoch images)
- **AMIsurvey** (End-to-end telescope-specific pipeline that ties the stages together, producing calibrated and cleaned multi-epoch images ready for source extraction and transient identification) — https://github.com/timstaley/amisurvey

## Evaluation signals

- The chimenea pipeline module runs end-to-end on the simulated measurement sets without errors
- Each epoch yields a primary-beam-corrected FITS image on disk
- Exported FITS images correspond one-to-one with input epochs of the multi-epoch measurement-set set
- Clean model components were placed only near sources in the deep-image mask, so exported epoch images are consistent with the masked-then-open Clean strategy
- Iterative background RMS re-estimation at each Clean pass was applied, indicating Clean-bias was avoided in the images being exported

## Limitations

- The exported FITS images are only produced for input file sizes and reduction times typical of ALARRM datasets (order of a gigabyte or less, reduction times on the order of minutes); re-Cleaning may become a significant processing-time contribution for larger datasets
- Fields with extended emission or complex blended sources still require manual reduction, so the automated per-epoch FITS export is not appropriate there
- If output-parsing relies on a particular external-tool output format, changes to that format can break the pipeline that produces the images to be exported

## Evidence

- [full_text] Export stage of chimenea dataflow: [evidence span withheld — anchor status: not_found]
- [intro] AMIsurvey pipeline purpose: [evidence span withheld — anchor status: not_found]
- [intro] chimenea uses CASA via drive-casa: [evidence span withheld — anchor status: not_found]
- [readme] chimenea pipeline location and CASA dependency: [evidence span withheld — anchor status: not_found]
