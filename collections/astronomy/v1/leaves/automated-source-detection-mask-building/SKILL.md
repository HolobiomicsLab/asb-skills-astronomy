---
name: automated-source-detection-mask-building
description: Use when imaging pre-calibrated multi-epoch radio-synthesis measurement
  sets with chimenea/CASA and you need a source mask so per-epoch Clean components
  are placed only near known steady sources.
when_to_use_negative:
- Fields containing extended emission or complex blended sources — the paper notes
  such fields 'still need to be manually reduced', so automated mask building from
  source detection is inappropriate
- Single-epoch data with no concatenated deep image available, since the mask must
  be derived from a deep image of steady sources
edam_operation: http://edamontology.org/operation_3646
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: chimenea
  role: Implements the pipeline logic-flow (pipeline.py) that runs automated source-detection
    on the deep image and builds the Clean mask image
  repo: https://github.com/timstaley/chimenea
- name: drive-casa
  role: Python interface layer used to invoke CASA Clean subroutines that generate
    the deep image on which source-detection and mask building operate
  repo: https://github.com/timstaley/drive-casa
- name: CASA
  role: Provides the underlying image-synthesis (Clean) subroutines used to produce
    the deep image and to apply the derived mask during per-epoch Cleans
- name: AMIsurvey
  role: End-to-end pipeline that ties the packages together, feeding calibrated multi-epoch
    data into the chimenea stage where mask building occurs
  repo: https://github.com/timstaley/amisurvey
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ascom.2015.08.003
    title: 'Chimenea and other tools: Automated imaging of multi-epoch radio-synthesis
      data with CASA'
  source_content_hash: sha256:5b6b016fd0f567caf324fe25e073be5d4b8a7567c7569ae4fcd60c47893eeca9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/timstaley/chimenea
  tools:
  - AMIsurvey
  - CASA
  - chimenea
  - drive-casa
  edam_operation: http://edamontology.org/operation_3646
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# automated-source-detection-mask-building

## Summary

Run automated source-detection on a deep image built from concatenated multi-epoch data and convert the detected steady-source components into a Clean mask image that constrains subsequent single-epoch deconvolution.

## When to use

Use when imaging pre-calibrated multi-epoch radio-synthesis measurement sets with chimenea/CASA and you need a source mask so per-epoch Clean components are placed only near known steady sources.

## When NOT to use

- Fields containing extended emission or complex blended sources — the paper notes such fields 'still need to be manually reduced', so automated mask building from source detection is inappropriate
- Single-epoch data with no concatenated deep image available, since the mask must be derived from a deep image of steady sources

## Inputs

- Deep image produced by concatenating all epochs and Cleaning with iterative background RMS re-estimation
- Pre-calibrated multi-epoch radio-synthesis measurement-set files
- CASA environment accessed via drive-casa

## Outputs

- Clean mask image built from detected source components
- Catalogue/detected source components on the deep image (used for mask construction)

## How to apply

First concatenate all epochs and run an open (unmasked) Clean via the CASA Clean task driven through drive-casa, re-running Clean iteratively while the background RMS is re-estimated at each pass to avoid Clean-bias. On the resulting deep image, run automated source-detection to identify steady sources across the field of view. Build a Clean mask image from the detected source components. This mask is then applied to each individual epoch, restricting deconvolution to the vicinity of known sources, before an open per-epoch Clean recovers any emission outside the mask. The mask-based approach is the mechanism by which the algorithm uses automated source-detection to avoid 'Clean-bias'.

## Related tools

- **chimenea** (Implements the pipeline logic-flow (pipeline.py) that runs automated source-detection on the deep image and builds the Clean mask image) — https://github.com/timstaley/chimenea
- **drive-casa** (Python interface layer used to invoke CASA Clean subroutines that generate the deep image on which source-detection and mask building operate) — https://github.com/timstaley/drive-casa
- **CASA** (Provides the underlying image-synthesis (Clean) subroutines used to produce the deep image and to apply the derived mask during per-epoch Cleans)
- **AMIsurvey** (End-to-end pipeline that ties the packages together, feeding calibrated multi-epoch data into the chimenea stage where mask building occurs) — https://github.com/timstaley/amisurvey

## Examples

```
from chimenea import pipeline; pipeline.reduce_all_fits_files_in_dir(casa_exec='/path/to/casa', obs_dir='obs/', output_dir='out/')
```

## Evaluation signals

- The mask image should contain Clean mask components only at positions of sources detected on the deep image of concatenated epochs
- Subsequent masked per-epoch Cleans should place Clean model components only in the vicinity of known (deep-image-detected) sources
- Running the pipeline end-to-end on simulated measurement sets should yield per-epoch images consistent with the paper's criterion of iterative RMS-estimation and automated source-detection avoiding Clean-bias
- The open (unmasked) per-epoch Clean initialized from the masked-epoch model should recover emission lying outside the mask

## Limitations

- No automated handling for extended emission or complex blended sources; such fields still need manual reduction
- Iterative re-Cleaning with RMS re-estimation can dominate processing time for large datasets
- Reduction was benchmarked only on small ALARRM datasets (order of a gigabyte or less, reduction times on the order of minutes)

## Evidence

- [intro] Deep-image mask derivation: [evidence span withheld — anchor status: not_found]
- [abstract] Clean-bias avoidance rationale: "The algorithm aims to improve upon standard imaging pipelines by utilizing iterative RMS-estimation and automated source-detection to avoid so called ‘Clean-bias’"
- [discussion] Iterative re-Clean loop: [evidence span withheld — anchor status: not_found]
- [readme] Pipeline key logic location: "The key logic-flow is implemented in the [pipeline.py](chimenea/pipeline.py) module."
