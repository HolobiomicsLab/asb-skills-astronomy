---
name: background-rms-estimation-via-sigma-clipping
description: Use when cleaning radio-interferometric dirty/residual maps where a noise
  threshold must be estimated from an image that is itself not yet clean, and Clean-bias
  must be avoided.
when_to_use_negative:
- The field contains extended emission or complex blended sources — the pipeline has
  no automated handling for these and such fields still need to be manually reduced
- A trustworthy a-priori RMS is already available from the visibility data, making
  iterative re-estimation unnecessary overhead
- Computational budget is tight for very large datasets — for a large dataset the
  re-Cleaning may become a significant contribution to the overall processing time
edam_operation: http://edamontology.org/operation_3443
tools:
- name: chimenea
  role: Implements the iterative re-Clean / RMS-estimation subroutine (key logic-flow
    in chimenea/pipeline.py)
  repo: https://github.com/timstaley/chimenea
- name: drive-casa
  role: Python interface layer through which the CASA Clean subroutine is invoked
    for each re-Clean iteration
  repo: https://github.com/timstaley/drive-casa
- name: CASA
  role: Provides the underlying image-synthesis Clean operation executed with the
    sigma-clipped RMS threshold
- name: Python
  role: Host language implementing sigma-clipping, bias correction, and the iteration/termination
    logic
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.08.003
    title: 'Chimenea and other tools: Automated imaging of multi-epoch radio-synthesis
      data with CASA'
  source_content_hash: sha256:6d985e7f4ce4a1fc9cc7459e7d81b838fd3fac4b4ec7ad56278efee097bd2b25
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/timstaley/chimenea
  tools:
  - CASA
  - Python
  - chimenea
  - drive-casa
  edam_operation: http://edamontology.org/operation_3443
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# background-rms-estimation-via-sigma-clipping

## Summary

Estimates the background RMS of a radio-synthesis residual map by sigma-clipping its pixel values, applying a bias correction for inter-pixel correlation introduced by the synthesized beam, and feeding the resulting threshold into an iterative re-Clean loop that avoids 'Clean-bias'.

## When to use

Use when cleaning radio-interferometric dirty/residual maps where a noise threshold must be estimated from an image that is itself not yet clean, and Clean-bias must be avoided.

## When NOT to use

- The field contains extended emission or complex blended sources — the pipeline has no automated handling for these and such fields still need to be manually reduced
- A trustworthy a-priori RMS is already available from the visibility data, making iterative re-estimation unnecessary overhead
- Computational budget is tight for very large datasets — for a large dataset the re-Cleaning may become a significant contribution to the overall processing time

## Inputs

- Pre-calibrated measurement set or dirty residual map from the input working directory
- User-defined Clean threshold expressed as a multiple of the estimated background RMS
- RMS-decrease tolerance (default 5%) and maximum iteration count (default 3)

## Outputs

- Converged sigma-clipped background RMS estimate
- Final restored cleaned image exported as reclean_image.fits
- Residual component model
- rms_iterations.csv logging the RMS value at each re-Clean iteration

## How to apply

Load the pre-calibrated measurement set or dirty residual map and set up the CASA toolkit via the drive-casa Python interface. Take the pixel values of the current residual map and apply sigma-clipping to estimate the background RMS, applying the bias correction for inter-pixel correlation introduced by the synthesized beam. Run the CASA Clean subroutine with its threshold set to a user-defined multiple of this estimated RMS. Re-compute the sigma-clipped, beam-correlation-corrected RMS on the new residual map; if the proportional decrease relative to the previous estimate is below the tolerance (default 5%) or the iteration count has reached the maximum (default 3), stop — otherwise log the RMS to rms_iterations.csv and re-Clean. Export the final restored image and residual component model to reclean_image.fits with the converged RMS recorded. The rationale is that RMS cannot be estimated a priori from an un-cleaned map, so estimation and cleaning must iterate until convergence.

## Related tools

- **chimenea** (Implements the iterative re-Clean / RMS-estimation subroutine (key logic-flow in chimenea/pipeline.py)) — https://github.com/timstaley/chimenea
- **drive-casa** (Python interface layer through which the CASA Clean subroutine is invoked for each re-Clean iteration) — https://github.com/timstaley/drive-casa
- **CASA** (Provides the underlying image-synthesis Clean operation executed with the sigma-clipped RMS threshold)
- **Python** (Host language implementing sigma-clipping, bias correction, and the iteration/termination logic)

## Examples

```
from chimenea import pipeline  # key logic-flow implemented in chimenea/pipeline.py
```

## Evaluation signals

- The proportional decrease in RMS between successive re-Clean iterations falls below the 5% tolerance, or 3 iterations have been completed
- Each iteration's RMS value is appended to rms_iterations.csv, giving an auditable convergence trace
- The final restored image and residual component model exist as reclean_image.fits with the converged RMS recorded
- The Clean threshold at every iteration equals the user-defined multiple of the beam-correlation-corrected sigma-clipped RMS computed from the current residual map
- The cleaned residual map shows no over-cleaning (Clean-bias) around source positions

## Limitations

- For a large dataset the re-Cleaning may become a significant contribution to the overall processing time
- Fields containing extended emission or complex blended sources still need to be manually reduced
- Reduction was benchmarked only on small (order of a gigabyte or less) ALARRM datasets

## Evidence

- [abstract] Iterative RMS estimation terminating on 5% tolerance / 3 iterations: "The algorithm aims to improve upon standard imaging pipelines by utilizing iterative RMS-estimation and automated source-detection to avoid so called ‘Clean-bias’"
- [discussion] Repeated Clean with re-estimated background RMS: [evidence span withheld — anchor status: not_found]
- [readme] chimenea implements the algorithm on multi-epoch radio-synthesis data: "Chimenea implements a simple but effective algorithm for automated imaging
of multi-epoch radio-synthesis data, developed as part of the 
[ALARRM programme]"
