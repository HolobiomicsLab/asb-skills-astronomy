---
name: 3d-model-deviation-comparison
description: Use when you have both a reconstructed 3-D model (from SfM/MVS over rendered
  or real images) and the original reference model, and need quantitative reconstruction-accuracy
  metrics such as Gaussian mean and standard deviation of point deviations.
when_to_use_negative:
- You only have rendered 2-D images and no reconstructed 3-D model or reference input
  model to compare against
- You need to compare two images rather than two 3-D models — the S1 script is an
  image-comparison algorithm used in the manuscript, not a general mesh-registration
  tool
- The target is a comet or active asteroid where SISPO's dust- and gas-environment
  models dominate, and shape-reconstruction accuracy metrics are not the goal
tools:
- name: SISPO
  role: Provides the full pipeline from simulated imagery (rendered orbiting images
    of the spinning SSSB) to final 3-D data products that are compared against the
    reference model
  repo: https://github.com/SISPO-developers/sispo
- name: Blender / Cycles
  role: Renders the physically based, photorealistic pinhole-camera images (53 viewing
    angles of the spinning target) from which the reconstructed model is derived
- name: OpenMVG
  role: Structure-from-motion on the rendered image set to recover camera poses and
    a sparse point cloud
- name: OpenMVS
  role: Multi-view stereo densification and meshing to produce the fully reconstructed
    3-D model compared to the reference
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pone.0263882
    title: 'SISPO: Space Imaging Simulator for Proximity Operations'
  source_content_hash: sha256:4f08c85b0ab3447122a2007c7cf11533e3a8f63ef27753452e73a7bc45889204
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SISPO-developers/sispo
  tools:
  - Blender / Cycles
  - OpenMVG
  - OpenMVS
  - SISPO
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# 3d-model-deviation-comparison

## Summary

Quantitatively compares a reconstructed 3-D model (e.g., an OpenMVS mesh from structure-from-motion) against the reference input 3-D model by computing per-point deviation statistics, as demonstrated in the SISPO SSSB orbiting case study on the 25143 Itokawa asteroid model.

## When to use

Use when you have both a reconstructed 3-D model (from SfM/MVS over rendered or real images) and the original reference model, and need quantitative reconstruction-accuracy metrics such as Gaussian mean and standard deviation of point deviations.

## When NOT to use

- You only have rendered 2-D images and no reconstructed 3-D model or reference input model to compare against
- You need to compare two images rather than two 3-D models — the S1 script is an image-comparison algorithm used in the manuscript, not a general mesh-registration tool
- The target is a comet or active asteroid where SISPO's dust- and gas-environment models dominate, and shape-reconstruction accuracy metrics are not the goal

## Inputs

- Reconstructed 3-D model from multi-view stereo (e.g., OpenMVS mesh produced from rendered or acquired images)
- Reference input 3-D model (e.g., the 25143 Itokawa asteroid model used by SISPO)
- Supplementary S1 image-comparison Python script
- Optionally the rendered image set (53 pinhole-camera images from different viewing angles) used to produce the reconstruction

## Outputs

- Per-point deviation statistics between reconstructed and reference models
- Gaussian mean error of model deviations (case-study value ~1.2 m)
- Standard deviation of model deviations (case-study value ~6.6 m)
- Spatially localized deviation report (e.g., crater-spot deviations of 60–89.7 m)

## How to apply

Run the supplementary S1 image-comparison Python algorithm to compare the reconstructed model against the input reference model, computing point-wise deviation statistics. In the SISPO orbiting case study, 53 pinhole-camera images rendered with Blender's Cycles path-tracing engine were processed through OpenMVG (SfM: camera poses and sparse point cloud) and OpenMVS (densification and meshing) to obtain the fully reconstructed 3-D model. The comparison against the 25143 Itokawa reference model yielded a Gaussian mean error of 1.2 m and a standard deviation of 6.6 m. Decision point: deviations localized at crater spots reaching 60–89.7 m are expected and acceptable; a large global mean/std indicates the image set (viewing angles, resolution) is insufficient. Validation is by reproducing the reported case-study statistics, and accuracy can be improved with more images and higher resolution.

## Related tools

- **SISPO** (Provides the full pipeline from simulated imagery (rendered orbiting images of the spinning SSSB) to final 3-D data products that are compared against the reference model) — https://github.com/SISPO-developers/sispo
- **Blender / Cycles** (Renders the physically based, photorealistic pinhole-camera images (53 viewing angles of the spinning target) from which the reconstructed model is derived)
- **OpenMVG** (Structure-from-motion on the rendered image set to recover camera poses and a sparse point cloud)
- **OpenMVS** (Multi-view stereo densification and meshing to produce the fully reconstructed 3-D model compared to the reference)

## Evaluation signals

- Computed Gaussian mean error of model deviations reproduces the reported ~1.2 m for the 53-image orbiting Itokawa case
- Computed standard deviation reproduces the reported ~6.6 m
- Residual large deviations are confined to crater spots and fall within the reported 60–89.7 m range
- Global mean and standard deviation do not blow up (a large global mean indicates an insufficient image set rather than expected localized artifacts)
- Increasing image count and resolution should reduce the deviation statistics, consistent with the article's stated improvement path

## Limitations

- Deviation reaches 60–89.7 m at some crater spots, so localized errors dominate even when global statistics are small
- Accuracy depends on image count and resolution; 53 images is sufficient for a full reconstruction but improving accuracy requires more images and higher resolution
- Using SISPO (and the surrounding pipeline) requires basic programming skills, which limits accessibility

## Evidence

- [supplementary] S1 ScriptImage comparison.The file contains python algorithm used to compare images in this manuscript.: "S1 ScriptImage comparison.The file contains python algorithm used to compare images in this manuscript."
- [intro] This article demonstrates the synthetic image simulation capabilities of SISPO, and 3-D reconstruction as a case study showing how such images can be utilised.: "This article demonstrates the synthetic image simulation capabilities of SISPO, and 3-D reconstruction as a case study showing how such images can be utilised."
- [discussion] The tool generates physically based, photorealistic images from an input 3-D model using Blender’s Cycles path-tracing rendering engine.: "The tool generates physically based, photorealistic images from an input 3-D model using Blender’s Cycles path-tracing rendering engine."
- [intro] The 25143 Itokawa asteroid model by the [14] was used.: "The 25143 Itokawa asteroid model by the [14] was used."
