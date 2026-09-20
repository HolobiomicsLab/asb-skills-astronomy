---
name: spacecraft-camera-model-configuration
description: Use when building a synthetic imaging pipeline for a small-body fly-by
  or proximity-operations scenario and camera intrinsics, trajectory, illumination,
  or renderer (OpenGL vs Cycles) must be set before image generation.
when_to_use_negative:
- The task is real image processing of acquired spacecraft photographs rather than
  synthetic image generation from a 3-D model.
- Photorealistic radiometric fidelity of already-rendered scenes is the goal and the
  OpenGL lightweight renderer's speed would compromise it without justification.
- Using ComaCreator as a standalone science tool — the README warns it is for realistic-looking
  pictures, not real science.
tools:
- name: SISPO
  role: Main imaging simulator whose camera model, trajectory, lighting, and renderer
    (OpenGL vs Cycles) are configured to generate the synthetic image sequence
  repo: https://github.com/SISPO-developers/sispo
- name: Blender/Cycles
  role: Photorealistic physically based path-tracing rendering engine invoked by SISPO
    when high-fidelity rendering is selected
  repo: github.com/SISPO-developers/sispo
- name: sispo_docker
  role: Docker image that simplifies installation and provides a ready environment
    for running SISPO (and ComaCreator plugins)
  repo: https://github.com/SISPO-developers/sispo_docker
- name: ComaCreator
  role: Optional dust/gas environment plugin activated through the SISPO input file's
    options and plugins keys before rendering (Cycles only)
  repo: SISPO-developers/ComaCreator
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pone.0263882
    title: 'SISPO: Space Imaging Simulator for Proximity Operations'
  source_content_hash: sha256:3ae3d184a6e20845ecbbb4e67d1b55f17f9a324ca1529f0f414d1089f32f2804
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SISPO-developers/sispo
  tools:
  - Blender/Cycles
  - ComaCreator
  - SISPO
  - sispo_docker
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Spacecraft camera model configuration

## Summary

Configure SISPO's spacecraft camera model — intrinsic parameters, trajectories, lighting, and renderer choice (lightweight OpenGL vs. Blender/Cycles path tracer) — to generate synthetic proximity-operation images of small solar system bodies such as 25143 Itokawa.

## When to use

Use when building a synthetic imaging pipeline for a small-body fly-by or proximity-operations scenario and camera intrinsics, trajectory, illumination, or renderer (OpenGL vs Cycles) must be set before image generation.

## When NOT to use

- The task is real image processing of acquired spacecraft photographs rather than synthetic image generation from a 3-D model.
- Photorealistic radiometric fidelity of already-rendered scenes is the goal and the OpenGL lightweight renderer's speed would compromise it without justification.
- Using ComaCreator as a standalone science tool — the README warns it is for realistic-looking pictures, not real science.

## Inputs

- 3-D shape model (e.g. the 25143 Itokawa asteroid model)
- SISPO input file specifying camera intrinsic parameters, trajectory, light parameters, and renderer choice
- ComaCreator plugin settings (mesh_file, decimate_percentage, JetCreator pipeline) when dust/gas environments are needed

## Outputs

- Synthetic image sequence for the fly-by/proximity scenario
- Rendered camera images from the OpenGL or Cycles renderer
- Per-image generation-time and mission-wall-clock measurements for benchmarking

## How to apply

Install sispo (pip install gives a CLI, or use the sispo_docker image) and load the target 3-D shape model — the paper uses the 25143 Itokawa asteroid model. Configure the camera's intrinsic parameters, spacecraft trajectory, and light parameters in the SISPO input, and select the renderer: the lightweight OpenGL renderer (~0.7–1.0 s per image, ~12 h per simulated mission week on an Intel i7-7700HQ 2.8 GHz) for closed-loop GNC and large batch simulation, versus the photorealistic Cycles path tracer when physically based radiometric fidelity matters. For cometary/active-asteroid scenes, enable the ComaCreator plugin via the input file options/plugins keys, remembering the default decimation of the shape mesh to ~1% (about 3000 of 3 million triangles) before rendering. Record per-image generation times and total wall-clock time when benchmarking, since renderer choice dominates runtime.

## Related tools

- **SISPO** (Main imaging simulator whose camera model, trajectory, lighting, and renderer (OpenGL vs Cycles) are configured to generate the synthetic image sequence) — https://github.com/SISPO-developers/sispo
- **Blender/Cycles** (Photorealistic physically based path-tracing rendering engine invoked by SISPO when high-fidelity rendering is selected) — github.com/SISPO-developers/sispo
- **sispo_docker** (Docker image that simplifies installation and provides a ready environment for running SISPO (and ComaCreator plugins)) — https://github.com/SISPO-developers/sispo_docker
- **ComaCreator** (Optional dust/gas environment plugin activated through the SISPO input file's options and plugins keys before rendering (Cycles only)) — SISPO-developers/ComaCreator

## Examples

```
{"options": ["--with-render","--with-sim"], "plugins": ["/app/ComaCreator/interface.py"], "ComaCreator": {"decimate_mesh": true, "decimate_percentage": 0.001}}
```

## Evaluation signals

- Rendered sequence uses the requested renderer (OpenGL for speed-critical GNC runs; Cycles when the physically based path tracer is required).
- Camera images of the loaded shape model (e.g. Itokawa) show the configured intrinsics — framing, trajectory, and illumination match the input file.
- Per-image generation time on i7-7700HQ-class hardware is on the order of 0.7–1.0 s for the OpenGL renderer; large deviations flag misconfiguration.
- Benchmark outputs (per-image times, mission-week wall clock ~12 h) tabulated consistently, e.g. in a render_benchmark.csv.
- When ComaCreator is enabled, the input file contains the --with-plugin option, a plugins path, and a decimate_percentage in [0,1] (e.g. 0.001 for ~3000 triangles).

## Limitations

- SISPO is open access but requires basic programming skills to use.
- The lightweight OpenGL renderer trades photorealism (Cycles path tracing) for speed; the two are not radiometrically equivalent.
- ComaCreator is in a development stage and should only be used for creating realistic-looking pictures, not real science.
- ComaCreator output (EXR) file sizes can grow to ~1 GB easily; iterating 3 million triangles without decimation 'will take forever'.

## Evidence

- [abstract] Camera intrinsics and renderer choice are first-class SISPO inputs: "SISPO allows the simulation of trajectories, light parameters and camera’s intrinsic parameters"
- [discussion] Renderer selection: Cycles path tracer for physically based images: "The tool generates physically based, photorealistic images from an input 3-D model using Blender’s Cycles path-tracing rendering engine."
- [methods] Installation path that yields the CLI used to drive rendering: "If sispo is installed as a python package via pip, sispo provides a CLI."
- [intro] Python-programmable Blender integration underlying camera/scene configuration: "SISPO uses Cycles rendering with the Blender software package, which allows for programming procedures in Python [15]."
- [methods] Docker alternative for setting up the SISPO rendering environment: "There is another repository linked to sispo which uses Docker to simplify the installation."
- [readme] Plugin activation and mesh decimation keys in the SISPO input file (README): ""options": ["--with-render","--with-sim", "--with-plugin"],
    "plugins": ["/app/ComaCreator/interface.py"]"
