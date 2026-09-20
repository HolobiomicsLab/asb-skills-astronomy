---
name: synthesized-beam-pixel-correlation-bias-correction
description: Use when estimating the background noise level of a dirty/cleaned residual
  map from radio-synthesis data (e.g. AMI-LA via chimenea) to set a Clean threshold,
  because cleaned-beam sidelobes correlate neighbouring pixels and deflate naive sigma-clipped
  RMS estimates.
when_to_use_negative:
- When the noise estimate is needed for an image whose pixels are statistically independent
  (no restoring-beam smoothing), the beam-correlation correction is unnecessary and
  would overestimate the RMS
- When the field contains extended or complex blended sources, which the paper notes
  still need manual reduction rather than automated RMS-based thresholds
- When performing uv-plane model fitting for flux-density measurement after source
  identification — that is a separate post-imaging step, not threshold estimation
edam_operation: http://edamontology.org/operation_2238
tools:
- name: chimenea
  role: Implements the iterative RMS-estimation / re-Clean logic, including the beam-correlation-bias-corrected
    sigma-clipped RMS estimator
  repo: github.com/timstaley/chimenea
- name: drive-casa
  role: Python interface used to call CASA Clean with the corrected RMS-derived threshold
    and to read back the residual map
  repo: github.com/timstaley/drive-casa
- name: CASA
  role: Provides the Clean subroutine and image-synthesis operations whose residual
    maps supply the pixel values for RMS estimation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.08.003
    title: 'Chimenea and other tools: Automated imaging of multi-epoch radio-synthesis
      data with CASA'
  source_content_hash: sha256:881e85ae9057f5c197afc0ceda418fd7ca945c827da6734a6ad790f4187a4baf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - CASA
  - chimenea
  - drive-casa
  edam_operation: http://edamontology.org/operation_2238
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Synthesized-beam pixel-correlation bias correction

## Summary

Corrects the sigma-clipped background RMS estimate of a radio-synthesis residual map for inter-pixel correlation introduced by the synthesized beam, so that the Clean threshold derived from it is not biased low (avoiding 'Clean-bias').

## When to use

Use when estimating the background noise level of a dirty/cleaned residual map from radio-synthesis data (e.g. AMI-LA via chimenea) to set a Clean threshold, because cleaned-beam sidelobes correlate neighbouring pixels and deflate naive sigma-clipped RMS estimates.

## When NOT to use

- When the noise estimate is needed for an image whose pixels are statistically independent (no restoring-beam smoothing), the beam-correlation correction is unnecessary and would overestimate the RMS
- When the field contains extended or complex blended sources, which the paper notes still need manual reduction rather than automated RMS-based thresholds
- When performing uv-plane model fitting for flux-density measurement after source identification — that is a separate post-imaging step, not threshold estimation

## Inputs

- Dirty or partially-Cleaned residual map (FITS) from a radio-synthesis observation (e.g. AMI-LA measurement set imaged via CASA)
- Synthesized beam parameters (clean beam) needed for the inter-pixel correlation bias correction

## Outputs

- Beam-correlation-corrected sigma-clipped RMS value used as the Clean threshold
- Per-iteration RMS values appended to rms_iterations.csv
- Converged final RMS recorded alongside reclean_image.fits

## How to apply

Take the pixel values of the current residual map and apply sigma-clipping to estimate the background RMS, applying the bias correction for inter-pixel correlation introduced by the synthesized beam. Feed the corrected RMS (times a user-defined multiple) as the Clean threshold via the drive-casa interface to CASA. After each Clean, re-compute the sigma-clipped, beam-correlation-corrected RMS on the new residual map; the iterative RMS re-estimation loop stops when the proportional decrease is below 5% or after 3 iterations. This bias correction is the reason the pipeline can use an image (the residual map) to estimate its own termination threshold without incurring Clean-bias.

## Related tools

- **chimenea** (Implements the iterative RMS-estimation / re-Clean logic, including the beam-correlation-bias-corrected sigma-clipped RMS estimator) — github.com/timstaley/chimenea
- **drive-casa** (Python interface used to call CASA Clean with the corrected RMS-derived threshold and to read back the residual map) — github.com/timstaley/drive-casa
- **CASA** (Provides the Clean subroutine and image-synthesis operations whose residual maps supply the pixel values for RMS estimation)

## Examples

```
from chimenea import pipeline; pipeline.clean_imaging(prepdict)  # runs re-Clean loop with beam-bias-corrected RMS in chimenea/pipeline.py
```

## Evaluation signals

- The corrected sigma-clipped RMS is larger than an uncorrected sigma-clip estimate on the same residual map (bias correction inflates the estimate)
- The re-Clean loop terminates within the criteria: proportional RMS decrease below the 5% tolerance or at most 3 iterations
- rms_iterations.csv contains one row per Clean iteration with monotonically non-increasing RMS values
- The Clean threshold equals the user-defined multiple of the corrected RMS, not of a raw sigma-clipped value
- Final products reclean_image.fits and the residual component model are exported with the converged RMS value recorded

## Limitations

- Computational overhead of the iterative re-Cleaning (re-estimating RMS each iteration) is unquantified for large datasets and may dominate processing time
- Reduction performance was benchmarked only on small ALARRM datasets (order of a gigabyte or less, minutes of runtime)
- Fields containing extended emission or complex blended sources still need manual reduction

## Evidence

- [methods] Sigma-clip the pixel values of the current residual map to estimate the background RMS, applying the bias correction for inter-pixel correlation introduced by the synthesized beam.: [evidence span withheld — anchor status: not_found]
- [other] The re-Clean subroutine takes pixel values from the residuals map, applies sigma-clipping to estimate the background RMS, applies Clean with the resulting calculated threshold.: [evidence span withheld — anchor status: not_found]
- [abstract] The algorithm aims to improve upon standard imaging pipelines by utilizing iterative RMS-estimation and automated source-detection to avoid so called 'Clean-bias': [evidence span withheld — anchor status: not_found]
