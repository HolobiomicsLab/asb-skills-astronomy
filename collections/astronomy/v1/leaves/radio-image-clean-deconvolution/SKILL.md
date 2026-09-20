---
name: radio-image-clean-deconvolution
description: Use when you have pre-calibrated radio-synthesis measurement sets (e.g.
  AMI-LA ALARRM datasets) and must produce cleaned epoch images whose background RMS
  is robustly estimated rather than assumed, or when naive fixed-threshold Clean would
  bias the residuals.
when_to_use_negative:
- Fields containing extended emission or complex blended sources — the paper notes
  such fields 'still need to be manually reduced'
- Raw (uncalibrated) AMI-LA datafiles — these must first go through the drive-ami/AMIsurvey
  calibration stages before Clean deconvolution
- Very large datasets where the re-Cleaning overhead dominates — the paper warns 'for
  a large dataset the re-Cleaning may become a significant contribution to the overall
  processing time'
tools:
- name: chimenea
  role: Implements the iterative re-Clean RMS-estimation/Clean subroutine and overall
    imaging algorithm; key logic in chimenea/pipeline.py
  repo: https://github.com/timstaley/chimenea
- name: drive-casa
  role: Python interface layer through which chimenea invokes the CASA Clean subroutines
  repo: https://github.com/timstaley/drive-casa
- name: CASA
  role: Provides the underlying image-synthesis Clean operations executed at each
    re-Clean iteration
- name: AMIsurvey
  role: End-to-end AMI-LA pipeline that ties together drive-ami, drive-casa and chimenea
    to produce calibrated and cleaned multi-epoch images
  repo: https://github.com/timstaley/amisurvey
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.08.003
    title: 'Chimenea and other tools: Automated imaging of multi-epoch radio-synthesis
      data with CASA'
  source_content_hash: sha256:5eb07e37541125d69b047760dc18756e0874761512ad4c67dd1665e6afcfa89e
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
yes_rate: 0.9333333333333333
n_claims: 15
---

# radio-image-clean-deconvolution

## Summary

Iterative Clean-based deconvolution of pre-calibrated multi-epoch radio-synthesis data, in which the Clean threshold is re-derived from sigma-clipped residual-map RMS (with synthesized-beam bias correction) until the RMS improvement falls below a 5% tolerance or 3 iterations are reached. This avoids 'Clean-bias' by letting the residuals map determine when deconvolution should stop.

## When to use

Use when you have pre-calibrated radio-synthesis measurement sets (e.g. AMI-LA ALARRM datasets) and must produce cleaned epoch images whose background RMS is robustly estimated rather than assumed, or when naive fixed-threshold Clean would bias the residuals.

## When NOT to use

- Fields containing extended emission or complex blended sources — the paper notes such fields 'still need to be manually reduced'
- Raw (uncalibrated) AMI-LA datafiles — these must first go through the drive-ami/AMIsurvey calibration stages before Clean deconvolution
- Very large datasets where the re-Cleaning overhead dominates — the paper warns 'for a large dataset the re-Cleaning may become a significant contribution to the overall processing time'

## Inputs

- Pre-calibrated multi-epoch radio-synthesis measurement sets (CASA .ms format)
- Dirty residual map of the field of view
- Clean mask image derived from automated source detection on the deep (all-epoch concatenated) image
- User-defined Clean threshold expressed as a multiple of the estimated background RMS

## Outputs

- reclean_image.fits — final restored cleaned image with converged RMS recorded
- Residual component model from the final Clean
- rms_iterations.csv — log of RMS values from successive re-Clean iterations
- Primary-beam-corrected per-epoch FITS images

## How to apply

Load the pre-calibrated measurement set or dirty residual map and drive CASA via the drive-casa Python interface. Take pixel values from the current residuals map, sigma-clip them to estimate the background RMS with the bias correction for inter-pixel correlation introduced by the synthesized beam, and run CASA Clean with the threshold set to a user-defined multiple of that RMS. Re-compute the sigma-clipped, beam-correlation-corrected RMS on the new residual map and compare to the previous estimate: stop if the proportional decrease is below the default 5% tolerance or the default maximum of 3 iterations is reached; otherwise append the RMS to rms_iterations.csv and re-Clean. Rationale: the residuals map itself determines the termination threshold, so deconvolution stops when additional components no longer reduce the residual RMS, avoiding Clean-bias. The converged-RMS Clean is applied both to the deep concatenated image and, masked and unmasked (open), to each individual epoch before primary-beam-corrected FITS export.

## Related tools

- **chimenea** (Implements the iterative re-Clean RMS-estimation/Clean subroutine and overall imaging algorithm; key logic in chimenea/pipeline.py) — https://github.com/timstaley/chimenea
- **drive-casa** (Python interface layer through which chimenea invokes the CASA Clean subroutines) — https://github.com/timstaley/drive-casa
- **CASA** (Provides the underlying image-synthesis Clean operations executed at each re-Clean iteration)
- **AMIsurvey** (End-to-end AMI-LA pipeline that ties together drive-ami, drive-casa and chimenea to produce calibrated and cleaned multi-epoch images) — https://github.com/timstaley/amisurvey

## Evaluation signals

- rms_iterations.csv records successive sigma-clipped RMS values, and the final iteration shows a proportional RMS decrease below the 5% tolerance or an iteration count capped at 3
- reclean_image.fits is produced together with the residual component model, and the converged RMS value is recorded
- Clean model components appear only in the vicinity of sources in the deep-image-derived mask during the masked per-epoch Clean
- Each epoch yields a primary-beam-corrected FITS image when the pipeline module runs end-to-end on simulated measurement sets

## Limitations

- Fields with extended emission or blended sources fall outside the automated scope and require manual reduction
- Computational overhead of iterative re-Cleaning is unquantified and may be significant for large datasets
- Reduction time was benchmarked only on small (order of a gigabyte or less) ALARRM datasets with reduction times on the order of minutes
- The RMS estimator carries a bias correction for inter-pixel correlation introduced by the synthesized beam, which must be applied consistently at every iteration

## Evidence

- [abstract] Iterative Clean with RMS-driven threshold and termination: [evidence span withheld — anchor status: not_found]
- [discussion] Repeated Clean passes as background RMS is re-estimated: [evidence span withheld — anchor status: not_found]
