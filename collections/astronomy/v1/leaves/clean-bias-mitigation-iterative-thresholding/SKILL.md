---
name: clean-bias-mitigation-iterative-thresholding
description: Use when producing cleaned images from pre-calibrated multi-epoch radio-synthesis
  measurement sets and you need a defensible, data-driven Clean stopping threshold
  instead of a fixed flux cutoff, to avoid Clean-bias.
when_to_use_negative:
- Fields containing extended emission or complex blended sources — the paper notes
  such fields still need to be manually reduced
- Very large datasets where the iterative re-Cleaning overhead is unacceptable — the
  paper warns re-Cleaning may become a significant contribution to overall processing
  time
- Raw AMI-LA visibility data that has not yet passed through calibration; this skill
  operates on pre-calibrated, imaged data
edam_operation: http://edamontology.org/operation_3443
tools:
- name: chimenea
  role: Implements the telescope-agnostic iterative RMS-estimation/re-Clean algorithm
    and the pipeline.py logic-flow that runs this subroutine
  repo: https://github.com/timstaley/chimenea
- name: drive-casa
  role: Python interface layer used to invoke the CASA Clean subroutine from the iterative
    thresholding loop
  repo: https://github.com/timstaley/drive-casa
- name: CASA
  role: Provides the underlying image-synthesis Clean subroutine executed at each
    iteration with the sigma-clipped threshold
- name: Python
  role: Host language for the sigma-clipping RMS estimator, convergence checks, and
    rms_iterations.csv bookkeeping
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.08.003
    title: 'Chimenea and other tools: Automated imaging of multi-epoch radio-synthesis
      data with CASA'
  source_content_hash: sha256:e5b7c586b5bba243974174c6b19bd5a51e41f9d9d252875bf3c1d91064cc340d
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
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# clean-bias-mitigation-iterative-thresholding

## Summary

An iterative re-Clean subroutine from the chimenea pipeline that estimates the residual-map background RMS via sigma-clipping (with a synthesized-beam inter-pixel correlation bias correction), cleans to a user-defined multiple of that RMS, and re-estimates until the RMS stops decreasing, to avoid 'Clean-bias' in automated radio imaging.

## When to use

Use when producing cleaned images from pre-calibrated multi-epoch radio-synthesis measurement sets and you need a defensible, data-driven Clean stopping threshold instead of a fixed flux cutoff, to avoid Clean-bias.

## When NOT to use

- Fields containing extended emission or complex blended sources — the paper notes such fields still need to be manually reduced
- Very large datasets where the iterative re-Cleaning overhead is unacceptable — the paper warns re-Cleaning may become a significant contribution to overall processing time
- Raw AMI-LA visibility data that has not yet passed through calibration; this skill operates on pre-calibrated, imaged data

## Inputs

- Pre-calibrated measurement set / dirty residual map from the input working directory
- User-defined multiple of the estimated background RMS used as the Clean threshold
- Convergence parameters: 5% proportional RMS-decrease tolerance, maximum 3 iterations

## Outputs

- reclean_image.fits (final restored cleaned image)
- Residual component model from the final Clean
- Converged RMS value
- rms_iterations.csv recording intermediate RMS estimates per iteration

## How to apply

Load the dirty residual map from the working directory and drive CASA via the drive-casa Python interface. Sigma-clip the pixel values of the current residual map to estimate the background RMS, applying the bias correction for inter-pixel correlation introduced by the synthesized beam. Run the CASA Clean subroutine with its cleaning threshold set to a user-defined multiple of the estimated background RMS, then recompute the sigma-clipped, beam-correlation-corrected RMS on the new residual map. Compare the new RMS to the previous estimate: stop if the proportional decrease is below the 5% tolerance or after 3 iterations; otherwise append the RMS value to rms_iterations.csv and Clean again. Finally export the restored image and residual component model to reclean_image.fits and record the converged RMS value.

## Related tools

- **chimenea** (Implements the telescope-agnostic iterative RMS-estimation/re-Clean algorithm and the pipeline.py logic-flow that runs this subroutine) — https://github.com/timstaley/chimenea
- **drive-casa** (Python interface layer used to invoke the CASA Clean subroutine from the iterative thresholding loop) — https://github.com/timstaley/drive-casa
- **CASA** (Provides the underlying image-synthesis Clean subroutine executed at each iteration with the sigma-clipped threshold)
- **Python** (Host language for the sigma-clipping RMS estimator, convergence checks, and rms_iterations.csv bookkeeping)

## Evaluation signals

- The sequence of RMS values in rms_iterations.csv shows a proportional decrease below the 5% tolerance (or the loop terminates at the 3-iteration cap), indicating convergence
- reclean_image.fits exists as a valid FITS file containing the final restored cleaned image
- The converged RMS value recorded at export matches the last sigma-clipped estimate computed on the final residual map
- Each Clean run in the loop used a threshold equal to the configured multiple of the beam-correlation-corrected sigma-clipped RMS from the preceding residual map

## Limitations

- For a large dataset the re-Cleaning may become a significant contribution to the overall processing time (unquantified computational overhead)
- Clean-bias avoidance depends on the sigma-clipped RMS estimator's bias correction for inter-pixel correlation from the synthesized beam; fields with extended/blended emission still require manual reduction
- Reduction was benchmarked only on small ALARRM datasets (order of a gigabyte or less) with reduction times on the order of minutes

## Evidence

- [abstract] Iterative thresholding rationale: [evidence span withheld — anchor status: not_found]
- [discussion] Repeated Clean with RMS re-estimation: [evidence span withheld — anchor status: not_found]
