---
name: multi-view-stereo-mesh-reconstruction
description: Use when a synthetic (or real) multi-angle image series of a body must
  be turned into a 3-D model, and the reconstructed model must be validated against
  a reference shape model via point-deviation statistics.
when_to_use_negative:
- Input is a single image or too few images — structure-from-motion/multi-view stereo
  requires multiple overlapping views from different angles; the paper's minimum validated
  sets are 25 (fly-by) and 53 (orbiting) images
- The goal is photorealistic image generation rather than 3-D model recovery — use
  SISPO's rendering capability alone without the OpenMVG/OpenMVS reconstruction chain
- Comet coma/dust-gas environment visualization is needed — that is ComaCreator's
  plugin role, not the reconstruction pipeline
tools:
- name: SISPO
  role: Space imaging simulator providing the full pipeline from simulated imagery
    (fly-by/orbiting image series) to final data products (3-D models)
  repo: https://github.com/SISPO-developers/sispo
- name: Blender
  role: Open-source 3-D visualisation system hosting the Cycles engine used to render
    the physically based input images
  repo: https://github.com/SISPO-developers/sispo
- name: Cycles
  role: Blender's path-tracing rendering engine that generates the photorealistic
    images fed to structure-from-motion
- name: OpenMVG
  role: Incremental structure-from-motion over the rendered image series to recover
    camera poses and a sparse point cloud
- name: OpenMVS
  role: Dense multi-view stereo densification and meshing on the SfM output to obtain
    the reconstructed 3-D model
- name: CloudCompare
  role: Computes point-to-mesh Gaussian distances between the input shape model and
    the reconstruction for error statistics
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1371/journal.pone.0263882
    title: 'SISPO: Space Imaging Simulator for Proximity Operations'
  source_content_hash: sha256:eae24f7bd395a7e19142914ddf34edcd921b7c04e9bed6221cfaf66d758526d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SISPO-developers/sispo
  tools:
  - Blender
  - CloudCompare
  - Cycles
  - OpenMVG
  - OpenMVS
  - SISPO
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# multi-view-stereo-mesh-reconstruction

## Summary

Reconstructs a dense 3-D mesh of a small solar system body from rendered pinhole-camera image series by running OpenMVG incremental structure-from-motion followed by OpenMVS multi-view stereo densification, then quantifying reconstruction error against the input shape model. This is the 3-D reconstruction case study enabled by SISPO's physically based image simulation pipeline.

## When to use

Use when a synthetic (or real) multi-angle image series of a body must be turned into a 3-D model, and the reconstructed model must be validated against a reference shape model via point-deviation statistics.

## When NOT to use

- Input is a single image or too few images — structure-from-motion/multi-view stereo requires multiple overlapping views from different angles; the paper's minimum validated sets are 25 (fly-by) and 53 (orbiting) images
- The goal is photorealistic image generation rather than 3-D model recovery — use SISPO's rendering capability alone without the OpenMVG/OpenMVS reconstruction chain
- Comet coma/dust-gas environment visualization is needed — that is ComaCreator's plugin role, not the reconstruction pipeline

## Inputs

- Pinhole-camera image series of the target (e.g. 25 fly-by or 53 orbiting images rendered by SISPO/Blender Cycles from the 25143 Itokawa asteroid shape model)
- Input 3-D shape model of the target (25143 Itokawa asteroid model)
- Camera intrinsics, trajectory, and light parameters configured for the fly-by or orbiting scenario

## Outputs

- Recovered camera poses and sparse point cloud from OpenMVG structure-from-motion
- Fully reconstructed dense 3-D model (mesh) from OpenMVS
- Point-deviation statistics between reconstructed and input model: Gaussian mean error, standard deviation, and worst-case deviations (e.g. mean 14.7 m / std 63.9 m for fly-by; mean 1.2 m / std 6.6 m with 60–89.7 m crater-spot deviations for orbiting; edge deviations up to 600 m)

## How to apply

Render (or collect) an image series with a pinhole camera around the target — e.g. 25 images for a fly-by or 53 images from different angles for an orbiting scenario around the spinning 25143 Itokawa model — using Blender's Cycles path-tracing engine inside SISPO with configured camera intrinsics, trajectory, and light parameters. Run incremental structure-from-motion with OpenMVG over the image set to recover camera poses and a sparse point cloud. Feed the SfM output into OpenMVS for dense multi-view stereo reconstruction and meshing to obtain the fully reconstructed 3-D model. Compare the reconstruction against the input reference model (e.g. point-to-mesh Gaussian distances in CloudCompare, or the supplementary S1 image-comparison python algorithm) to compute Gaussian error statistics and flag worst-case deviations. Decision point: reconstruction accuracy improves with more images and higher resolution; expect systematically worse results for sparse fly-by sets than orbiting sets.

## Related tools

- **SISPO** (Space imaging simulator providing the full pipeline from simulated imagery (fly-by/orbiting image series) to final data products (3-D models)) — https://github.com/SISPO-developers/sispo
- **Blender** (Open-source 3-D visualisation system hosting the Cycles engine used to render the physically based input images) — https://github.com/SISPO-developers/sispo
- **Cycles** (Blender's path-tracing rendering engine that generates the photorealistic images fed to structure-from-motion)
- **OpenMVG** (Incremental structure-from-motion over the rendered image series to recover camera poses and a sparse point cloud)
- **OpenMVS** (Dense multi-view stereo densification and meshing on the SfM output to obtain the reconstructed 3-D model)
- **CloudCompare** (Computes point-to-mesh Gaussian distances between the input shape model and the reconstruction for error statistics)

## Evaluation signals

- Fly-by case (25 pinhole images): reconstructed model should give Gaussian mean error ≈ 14.7 m and standard deviation ≈ 63.9 m vs the input Itokawa model, with worst-case edge deviations up to ~600 m
- Orbiting case (53 images from different angles): Gaussian mean error ≈ 1.2 m and standard deviation ≈ 6.6 m, with large deviations (60–89.7 m) localized at crater spots
- The stats output must contain Gaussian mean and standard deviation fields (e.g. a stats CSV with mean and standard deviation columns)
- Reconstruction error should decrease when more images and higher resolution are used (orbiting 53-image set outperforms fly-by 25-image set)
- OpenMVG must return both camera poses and a sparse point cloud before OpenMVS densification can proceed

## Limitations

- Reconstruction accuracy degrades with fewer images: the 25-image fly-by set yields mean 14.7 m / std 63.9 m, roughly an order of magnitude worse than the 53-image orbiting set
- Deviation is not uniform — worst-case errors concentrate at crater spots (60–89.7 m in the orbiting case) and model edges (up to 600 m in the fly-by case)
- Accuracy can only be improved with more images and higher resolution
- SISPO is open access but currently requires basic programming skills to be used

## Evidence

- [abstract] SISPO provides a full pipeline from simulated imagery to 3-D reconstruction: "The image simulator utilises the open-source 3-D visualisation system Blender and its Cycles rendering engine, which supports physically based rendering capabilities"
- [intro] SISPO developed to provide full pipeline to final data products such as 3-D models: "The Space Imaging Simulator for Proximity Operations (SISPO) has been developed to provide a full pipeline from simulated imagery to final data products (e.g., 3-D models)"
- [supplementary] Supplementary S1 python algorithm used to compare images: "S1 ScriptImage comparison.The file contains python algorithm used to compare images in this manuscript."
- [methods] SISPO CLI available via pip installation: "If sispo is installed as a python package via pip, sispo provides a CLI."
