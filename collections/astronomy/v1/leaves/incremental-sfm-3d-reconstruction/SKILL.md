---
name: incremental-sfm-3d-reconstruction
description: Use when a synthetic fly-by image sequence (e.g., 25 pinhole-camera images
  rendered with SISPO/Blender Cycles from a known shape model such as 25143 Itokawa)
  must be turned into camera poses and a 3-D model whose deviation from the ground-truth
  model is reported as Gaussian error statistics.
when_to_use_negative:
- The goal is only photorealistic image generation (SISPO rendering with Cycles) rather
  than recovering a 3-D model from those images
- The image set provides insufficient multi-view overlap or baseline for structure-from-motion
  (e.g., a single view or very few images)
- The task is comet coma/dust-environment optical modeling via ComaCreator rather
  than solid-body 3-D reconstruction
tools:
- name: SISPO
  role: Generates the physically based fly-by image series and provides a full pipeline
    from simulated imagery to final data products such as 3-D models
  repo: github.com/SISPO-developers/sispo
- name: Blender
  role: 3-D visualisation system hosting the rendering used to produce the SfM input
    images
  repo: https://github.com/SISPO-developers/sispo
- name: Cycles
  role: Blender's path-tracing engine used for physically based rendering of the fly-by
    images
  repo: https://github.com/SISPO-developers/sispo
- name: sispo_docker
  role: Docker image recommended as the easiest way to deploy SISPO and reproduce
    the image-generation step
  repo: https://github.com/SISPO-developers/sispo_docker
- name: OpenMVG
  role: Incremental structure-from-motion over the 25-image series to recover camera
    poses and a sparse point cloud (per the source workflow card)
- name: OpenMVS
  role: Dense multi-view stereo on the SfM output to obtain the dense 3-D reconstruction
    (per the source workflow card)
- name: CloudCompare
  role: Computes point-to-mesh Gaussian distances between the input shape model and
    the reconstructed model (per the source workflow card)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pone.0263882
    title: 'SISPO: Space Imaging Simulator for Proximity Operations'
  source_content_hash: sha256:422049eb724a2e0eb471b0adb092d94b1bffd7bf6aba1378140d82f6a146e42e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Blender
  - CloudCompare
  - Cycles
  - OpenMVG
  - OpenMVS
  - SISPO
  - sispo_docker
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# incremental-sfm-3d-reconstruction

## Summary

Reconstructs a 3-D model of a small solar system body from a short (25-image) SISPO-rendered fly-by image series via incremental structure-from-motion (OpenMVG) plus dense multi-view stereo (OpenMVS), then quantifies reconstruction accuracy as point-to-mesh Gaussian distances against the input shape model.

## When to use

Use when a synthetic fly-by image sequence (e.g., 25 pinhole-camera images rendered with SISPO/Blender Cycles from a known shape model such as 25143 Itokawa) must be turned into camera poses and a 3-D model whose deviation from the ground-truth model is reported as Gaussian error statistics.

## When NOT to use

- The goal is only photorealistic image generation (SISPO rendering with Cycles) rather than recovering a 3-D model from those images
- The image set provides insufficient multi-view overlap or baseline for structure-from-motion (e.g., a single view or very few images)
- The task is comet coma/dust-environment optical modeling via ComaCreator rather than solid-body 3-D reconstruction

## Inputs

- Fly-by image series (25 images in the reference case) rendered by SISPO with Blender Cycles using a pinhole camera model
- Reference/ground-truth 3-D shape model (25143 Itokawa asteroid model in the reference case)
- Camera intrinsic parameters, trajectory, and light-parameter configuration for the fly-by scenario

## Outputs

- Recovered camera poses for the image series
- Sparse point cloud from incremental structure-from-motion (OpenMVG)
- Dense 3-D reconstruction from multi-view stereo (OpenMVS)
- Gaussian error statistics (mean 14.7 m, std 63.9 m in the reference fly-by case) and flagged worst-case edge deviations (up to ~600 m)
- Stats CSV containing mean and standard-deviation fields

## How to apply

First render the fly-by series with SISPO using Blender's Cycles path-tracing engine, configuring pinhole camera intrinsics, trajectory, and light parameters (the paper's case uses the 25143 Itokawa shape model and a 25-image set). Run incremental structure-from-motion (OpenMVG) over the 25-image series to recover camera poses and a sparse point cloud, then run dense multi-view stereo (OpenMVS) on the SfM output to obtain a dense 3-D reconstruction. Load both the input shape model and the reconstructed model into CloudCompare and compute point-to-mesh Gaussian distances between them. Summarize deviations as Gaussian mean and standard deviation and flag worst-case deviations, which in the published fly-by case occur at model edges (up to ~600 m). The published reference result is a Gaussian mean error of 14.7 m with a standard deviation of 63.9 m, so the stats CSV must contain explicit mean and standard-deviation fields to allow validation against the article values.

## Related tools

- **SISPO** (Generates the physically based fly-by image series and provides a full pipeline from simulated imagery to final data products such as 3-D models) — github.com/SISPO-developers/sispo
- **Blender** (3-D visualisation system hosting the rendering used to produce the SfM input images) — https://github.com/SISPO-developers/sispo
- **Cycles** (Blender's path-tracing engine used for physically based rendering of the fly-by images) — https://github.com/SISPO-developers/sispo
- **sispo_docker** (Docker image recommended as the easiest way to deploy SISPO and reproduce the image-generation step) — https://github.com/SISPO-developers/sispo_docker
- **OpenMVG** (Incremental structure-from-motion over the 25-image series to recover camera poses and a sparse point cloud (per the source workflow card))
- **OpenMVS** (Dense multi-view stereo on the SfM output to obtain the dense 3-D reconstruction (per the source workflow card))
- **CloudCompare** (Computes point-to-mesh Gaussian distances between the input shape model and the reconstructed model (per the source workflow card))

## Examples

```
sispo --with-render --with-sim
```

## Evaluation signals

- The stats CSV contains explicit mean and standard-deviation fields
- Recomputed Gaussian error statistics are consistent with the published fly-by case values (mean 14.7 m, std 63.9 m)
- Worst-case deviations are flagged at model edges and reach up to ~600 m
- SfM recovers camera poses and a sparse point cloud for the full 25-image series before dense MVS is run
- Both the input shape model and the reconstructed model are loaded and compared via point-to-mesh Gaussian distances (e.g., in CloudCompare)

## Limitations

- Using SISPO (and hence this pipeline) currently requires basic programming skills, limiting accessibility
- Reconstruction errors are not uniform: worst-case deviations appear at model edges (up to ~600 m) even though the mean error is only 14.7 m
- No changelog found for the source repository, so version provenance of the tooling is limited

## Evidence

- [intro] SISPO full pipeline to 3-D data products: "The Space Imaging Simulator for Proximity Operations (SISPO) has been developed to provide a full pipeline from simulated imagery to final data products (e.g., 3-D models)"
- [abstract] Camera/trajectory/light simulation capability: "SISPO allows the simulation of trajectories, light parameters and camera’s intrinsic parameters"
- [discussion] Image generation via Cycles path-tracing: "The tool generates physically based, photorealistic images from an input 3-D model using Blender’s Cycles path-tracing rendering engine."
- [methods] SISPO CLI installation: "If sispo is installed as a python package via pip, sispo provides a CLI."
