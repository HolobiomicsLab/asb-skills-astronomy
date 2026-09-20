---
name: pinhole-camera-intrinsic-configuration
description: Use when generating synthetic imaging data for computer-vision benchmarking
  (e.g. SfM/3-D reconstruction) from a rendered fly-by scenario, and you must specify
  camera intrinsic parameters and trajectory/light settings so reconstructed poses
  and geometry can be compared against ground truth.
when_to_use_negative:
- You need cometary dust/gas coma rendering realism for actual science, not imaging-algorithm
  benchmarking — ComaCreator itself warns it 'should be only used for creating pretty
  realistic looking pictures'
- Your imagery already comes from a real instrument with calibrated intrinsics and
  you do not need synthetic pinhole rendering
- The task is 3-D reconstruction itself (SfM/MVS) rather than configuring the imaging
  simulation that feeds it
tools:
- name: SISPO
  role: Simulates trajectories, light parameters and camera intrinsic parameters and
    provides the full pipeline from simulated imagery to data products
  repo: https://github.com/SISPO-developers/sispo
- name: Blender
  role: Hosts the camera/light/scene configuration; SISPO drives it programmatically
    in Python
- name: Cycles
  role: Physically based path-tracing rendering engine that produces the pinhole-camera
    fly-by images
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pone.0263882
    title: 'SISPO: Space Imaging Simulator for Proximity Operations'
  source_content_hash: sha256:d5cf421733c2b5b7822a6e19c7d2f2dcbe2753e902471cfbbd0aa642dc582ee0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SISPO-developers/sispo
  tools:
  - Blender
  - Cycles
  - SISPO
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# pinhole-camera-intrinsic-configuration

## Summary

Configure pinhole camera intrinsics (focal length, image resolution, distortion-free projection) when rendering synthetic fly-by image sequences in SISPO/Blender, so that the resulting images can be fed to structure-from-motion pipelines such as OpenMVG/OpenMVS for 3-D reconstruction of small solar system bodies.

## When to use

Use when generating synthetic imaging data for computer-vision benchmarking (e.g. SfM/3-D reconstruction) from a rendered fly-by scenario, and you must specify camera intrinsic parameters and trajectory/light settings so reconstructed poses and geometry can be compared against ground truth.

## When NOT to use

- You need cometary dust/gas coma rendering realism for actual science, not imaging-algorithm benchmarking — ComaCreator itself warns it 'should be only used for creating pretty realistic looking pictures'
- Your imagery already comes from a real instrument with calibrated intrinsics and you do not need synthetic pinhole rendering
- The task is 3-D reconstruction itself (SfM/MVS) rather than configuring the imaging simulation that feeds it

## Inputs

- Input 3-D shape model (e.g. the 25143 Itokawa asteroid model)
- Fly-by trajectory definition (spacecraft position/orientation over the encounter)
- Pinhole camera intrinsic parameters (focal length, principal point, image resolution)
- Light-source parameters for the encounter scene
- SISPO input configuration file (defaults.in style, e.g. with 'options' entries)

## Outputs

- Rendered fly-by image series (e.g. 25 pinhole-camera images) suitable for SfM input
- Known ground-truth camera intrinsics and poses for reconstruction-accuracy evaluation
- Downstream dense 3-D point cloud after OpenMVG/OpenMVS with error statistics (Gaussian mean/std of point-to-mesh distances)

## How to apply

Set the camera intrinsic parameters (pinhole model, focal length, image size) together with the fly-by trajectory and light parameters in the SISPO input configuration, since SISPO allows the simulation of trajectories, light parameters and camera's intrinsic parameters. Render the image series with Blender's Cycles path-tracing engine from an input 3-D shape model (e.g. the 25143 Itokawa model), keeping the intrinsics fixed and known so they can serve as ground truth or calibration priors for downstream SfM. In the published fly-by case, 25 pinhole-camera images rendered this way were sufficient for OpenMVG incremental SfM followed by OpenMVS dense multi-view stereo to recover a 3-D model with Gaussian mean error of 14.7 m and standard deviation of 63.9 m. Run sispo via its CLI (install as a Python package via pip, or use the sispo_docker image) with render/sim options enabled; if using the ComaCreator plugin for comet-like targets, add --with-plugin and the plugins path. Because Cycles is physically based, the rendered images include realistic illumination, which is what makes the intrinsic configuration meaningful for validating computer-vision algorithms.

## Related tools

- **SISPO** (Simulates trajectories, light parameters and camera intrinsic parameters and provides the full pipeline from simulated imagery to data products) — https://github.com/SISPO-developers/sispo
- **Blender** (Hosts the camera/light/scene configuration; SISPO drives it programmatically in Python)
- **Cycles** (Physically based path-tracing rendering engine that produces the pinhole-camera fly-by images)

## Examples

```
sispo --with-render --with-sim
```

## Evaluation signals

- The rendered image series (e.g. 25 images) reconstructs in OpenMVG/OpenMVS with Gaussian mean error ≈ 14.7 m and std ≈ 63.9 m relative to the input Itokawa shape model
- Worst-case deviations are confined to model edges (up to ~600 m), indicating intrinsics/poses were consistent enough for registration
- SISPO runs without errors from the CLI with the configured options (e.g. --with-render, --with-sim) and produces the expected rendered frames
- The stats CSV contains mean and standard deviation fields for the point-to-mesh Gaussian distance comparison in CloudCompare

## Limitations

- SISPO is open access but currently requires basic programming skills to be used
- ComaCreator-style plugins are only supported with the Cycles render engine and the standalone version is untested; use sispo_docker for a ready environment
- Reconstruction errors concentrate at model edges, so intrinsic/pose validation should not rely on edge regions alone

## Evidence

- [abstract] SISPO allows the simulation of trajectories, light parameters and camera's intrinsic parameters: "SISPO allows the simulation of trajectories, light parameters and camera’s intrinsic parameters"
- [discussion] Cycles path-tracing renders photorealistic images from the input 3-D model: "The tool generates physically based, photorealistic images from an input 3-D model using Blender’s Cycles path-tracing rendering engine."
- [methods] sispo CLI availability after pip install: "If sispo is installed as a python package via pip, sispo provides a CLI."
- [methods] Docker installation route for sispo environment: "There is another repository linked to sispo which uses Docker to simplify the installation."
- [readme] README: options list controlling rendering and simulation stages: [evidence span withheld — anchor status: external_doc]
