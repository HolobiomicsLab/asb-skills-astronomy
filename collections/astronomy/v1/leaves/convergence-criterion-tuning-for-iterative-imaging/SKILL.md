---
name: convergence-criterion-tuning-for-iterative-imaging
description: Use when automating radio-synthesis imaging (e.g. chimenea/CASA Clean)
  and you must decide when iterative re-Cleaning has converged, since estimating an
  RMS threshold requires an image to sample.
when_to_use_negative:
- The field contains extended emission or complex blended sources — the paper notes
  such fields 'still need to be manually reduced'
- A single Clean pass with a fixed threshold is acceptable, and the computational
  cost of repeated re-Cleaning on a large dataset is not justified
- The input is not a radio-synthesis dirty/residual map but an already-restored image,
  so no residual-based RMS re-estimation is meaningful
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3366
tools:
- name: chimenea
  role: Implements the iterative re-Clean RMS-estimation algorithm and convergence
    logic (key logic in chimenea/pipeline.py)
  repo: https://github.com/timstaley/chimenea
- name: drive-casa
  role: Python interface layer through which chimenea invokes the CASA Clean subroutine
    and toolkit
  repo: https://github.com/timstaley/drive-casa
- name: CASA
  role: Provides the Clean image-synthesis subroutine executed with each newly computed
    RMS threshold
- name: Python
  role: Hosts the drive-casa interface and the sigma-clipping / convergence-check
    logic
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.08.003
    title: 'Chimenea and other tools: Automated imaging of multi-epoch radio-synthesis
      data with CASA'
  source_content_hash: sha256:7f0fbf6b1b8ffa34c643f2b81edd6723e7f386ff6a6c26af6f26c857564d7f7a
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
  edam_topics:
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# convergence-criterion-tuning-for-iterative-imaging

## Summary

Tunes the stopping criteria for iterative re-Cleaning in the chimenea pipeline: a sigma-clipped, beam-correlation-corrected background RMS is re-estimated after each Clean pass, and iterations stop when the proportional RMS decrease falls below a 5% tolerance or a maximum of 3 iterations is reached.

## When to use

Use when automating radio-synthesis imaging (e.g. chimenea/CASA Clean) and you must decide when iterative re-Cleaning has converged, since estimating an RMS threshold requires an image to sample.

## When NOT to use

- The field contains extended emission or complex blended sources — the paper notes such fields 'still need to be manually reduced'
- A single Clean pass with a fixed threshold is acceptable, and the computational cost of repeated re-Cleaning on a large dataset is not justified
- The input is not a radio-synthesis dirty/residual map but an already-restored image, so no residual-based RMS re-estimation is meaningful

## Inputs

- Pre-calibrated measurement set or dirty residual map from the input working directory
- User-defined Clean threshold as a multiple of the estimated background RMS
- Tolerance on proportional RMS decrease (default 5%)
- Maximum number of re-Clean iterations (default 3)

## Outputs

- reclean_image.fits containing the final restored cleaned image and residual component model
- rms_iterations.csv logging the RMS value from each re-Clean iteration
- Converged background RMS value

## How to apply

Load the pre-calibrated measurement set / dirty residual map via the drive-casa Python interface to the CASA toolkit, then sigma-clip the pixel values of the current residual map to estimate the background RMS, applying the bias correction for inter-pixel correlation introduced by the synthesized beam. Run CASA Clean with a threshold set to a user-defined multiple of that estimated RMS. Re-compute the sigma-clipped RMS on the new residual map with the same beam-correlation-corrected estimator and compare it with the previous estimate: if the proportional decrease is below the default 5% tolerance, or the iteration count has reached the default maximum of 3, stop. Otherwise append the RMS value to rms_iterations.csv and Clean again; export the final restored image and residual component model to reclean_image.fits with the converged RMS recorded. The 5%/3-iteration criteria exist to avoid 'Clean-bias' by ensuring the cleaning threshold tracks the true residual noise rather than an initial, dirty-map estimate.

## Related tools

- **chimenea** (Implements the iterative re-Clean RMS-estimation algorithm and convergence logic (key logic in chimenea/pipeline.py)) — https://github.com/timstaley/chimenea
- **drive-casa** (Python interface layer through which chimenea invokes the CASA Clean subroutine and toolkit) — https://github.com/timstaley/drive-casa
- **CASA** (Provides the Clean image-synthesis subroutine executed with each newly computed RMS threshold)
- **Python** (Hosts the drive-casa interface and the sigma-clipping / convergence-check logic)

## Evaluation signals

- rms_iterations.csv records one RMS entry per Clean iteration, confirming each pass ran with a threshold derived from the previous residual map
- The loop terminates only when the proportional RMS decrease between consecutive iterations is below 5% OR the iteration count reaches 3 — check the logged sequence satisfies this invariant
- reclean_image.fits is produced containing both the final restored cleaned image and the residual component model
- The final cleaning threshold equals the user-defined multiple of the last sigma-clipped, beam-correlation-corrected RMS estimate
- The converged RMS recorded on export matches the last entry in rms_iterations.csv

## Limitations

- For a large dataset the re-Cleaning may become a significant contribution to the overall processing time; the overhead of iterative re-Cleaning is unquantified for large datasets
- Reduction was benchmarked only on small ALARRM datasets, of order a gigabyte or less, with reduction times on the order of minutes
- Fields containing extended emission or complex blended sources still need to be manually reduced

## Evidence

- [abstract] RMS estimation drives Clean termination: [evidence span withheld — anchor status: not_found]
- [discussion] Repeated Clean with RMS re-estimation: [evidence span withheld — anchor status: not_found]
