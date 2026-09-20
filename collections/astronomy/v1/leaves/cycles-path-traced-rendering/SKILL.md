---
name: cycles-path-traced-rendering
description: Use when you need synthetic imagery of a small solar system body (e.g.
  25143 Itokawa) or planet surface for instrument characterisation, mission planning
  or computer-vision algorithm development, and you have a 3-D shape model plus trajectory,
  illumination and camera parameters.
when_to_use_negative:
- When the target is a near-complete mystery before close encounter and no shape model,
  trajectory or camera parameters exist — SISPO's rendering requires these inputs.
- When a non-photorealistic or analytical radiance transfer is needed instead of physically
  based rendering; the article contrasts SISPO with alternatives such as SurRender
  and PANGU but Cycles path tracing is not a substitute for purpose-built instrument
  simulators.
- 'When no programming environment is available: the tool is open access but currently
  requires basic programming skills to be used.'
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: SISPO
  role: The imaging simulator environment that wraps the Cycles path-tracing rendering;
    drives trajectory, light and camera-intrinsic simulation and provides the CLI.
  repo: https://github.com/SISPO-developers/sispo
- name: Blender
  role: The open-source 3-D visualisation system through which the scene is loaded
    and scripted (Python API) before rendering.
  repo: https://github.com/SISPO-developers/sispo
- name: Cycles
  role: Blender's physically based path-tracing rendering engine that actually produces
    the photorealistic images.
- name: ComaCreator
  role: Optional SISPO plugin generating dust- and gas-environment optical models
    for comets and active asteroids; invoked before rendering with --with-render --with-sim
    --with-plugin.
- name: sispo_docker
  role: Docker image used to deploy SISPO quickly with a preconfigured environment
    including ComaCreator support.
  repo: https://github.com/SISPO-developers/sispo_docker
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pone.0263882
    title: 'SISPO: Space Imaging Simulator for Proximity Operations'
  source_content_hash: sha256:75b8a91a08c6611c91844fbe7f19a95753ea9e4c6db1c30944a40ff0b52553fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SISPO-developers/sispo
  tools:
  - Blender
  - ComaCreator
  - Cycles
  - SISPO
  - sispo_docker
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# cycles-path-traced-rendering

## Summary

Renders physically based, photorealistic images from an input 3-D asteroid/space-scene model using Blender's Cycles path-tracing engine inside SISPO, simulating trajectories, light parameters and camera intrinsics. It is the core rendering mechanism of the SISPO imaging-simulation pipeline for small-body fly-by and terrestrial-planet surface missions.

## When to use

Use when you need synthetic imagery of a small solar system body (e.g. 25143 Itokawa) or planet surface for instrument characterisation, mission planning or computer-vision algorithm development, and you have a 3-D shape model plus trajectory, illumination and camera parameters.

## When NOT to use

- When the target is a near-complete mystery before close encounter and no shape model, trajectory or camera parameters exist — SISPO's rendering requires these inputs.
- When a non-photorealistic or analytical radiance transfer is needed instead of physically based rendering; the article contrasts SISPO with alternatives such as SurRender and PANGU but Cycles path tracing is not a substitute for purpose-built instrument simulators.
- When no programming environment is available: the tool is open access but currently requires basic programming skills to be used.

## Inputs

- Plain 3-D mesh of the target body (e.g. the 25143 Itokawa asteroid shape model)
- Trajectory definition (spacecraft position over time for fly-by or surface mission)
- Light parameters (illumination conditions)
- Camera intrinsic parameters
- Optional dust/gas environment configuration for comets and active asteroids (ComaCreator plugin, e.g. comet mesh_file, n_particles, sample_points, bbox bounds)
- Optional procedural surface parameters (height-map weights, masks, albedo texture)

## Outputs

- Physically based, photorealistic rendered images of the scene (e.g. procedural_surface_render.png)
- Rendered frames suitable as input to SISPO's downstream data products (e.g. 3-D reconstruction)

## How to apply

Load the plain input 3-D mesh (e.g. the 25143 Itokawa asteroid model) into Blender through its Python scripting API, then configure Cycles as the rendering engine. Define the fly-by trajectory, illumination conditions and camera intrinsic parameters in the SISPO configuration, so that images are physically based rather than artistically lit. Optionally enable supplementary dust- and gas-environment models (via the ComaCreator plugin) for comets and active asteroids, and apply common image aberrations (tangential and sagittal astigmatism, internal and external comatic aberration, geometric distortions). For regolith realism, add procedural surface texturing with height maps for craters, rocks and sand, blended with per-map weights and masks, and derive a matching asteroid-albedo texture. Render with Cycles path tracing and save the output images; validate qualitatively against the paper's demonstration figures.

## Related tools

- **SISPO** (The imaging simulator environment that wraps the Cycles path-tracing rendering; drives trajectory, light and camera-intrinsic simulation and provides the CLI.) — https://github.com/SISPO-developers/sispo
- **Blender** (The open-source 3-D visualisation system through which the scene is loaded and scripted (Python API) before rendering.) — https://github.com/SISPO-developers/sispo
- **Cycles** (Blender's physically based path-tracing rendering engine that actually produces the photorealistic images.)
- **ComaCreator** (Optional SISPO plugin generating dust- and gas-environment optical models for comets and active asteroids; invoked before rendering with --with-render --with-sim --with-plugin.)
- **sispo_docker** (Docker image used to deploy SISPO quickly with a preconfigured environment including ComaCreator support.) — https://github.com/SISPO-developers/sispo_docker

## Examples

```
pip install sispo; sispo --with-render --with-sim --with-plugin --plugins /app/ComaCreator/interface.py
```

## Evaluation signals

- Verify the rendered output image (e.g. procedural_surface_render.png) exists and is a valid renderable image.
- Qualitatively compare the render against the paper's demonstration figures (Figs 3, 4, 7, 8) for the procedural surface shader.
- Confirm that the rendered frame reflects the configured trajectory, illumination and camera intrinsics rather than default artistic lighting — the images must be physically based.
- If image aberrations were enabled, check that tangential/sagittal astigmatism, comatic aberration and geometric distortions are visible per configuration.
- If the ComaCreator plugin was used, confirm SISPO invoked JetCreator automatically before rendering and that the EXR dust-particle field was produced without runtime errors.

## Limitations

- SISPO is open access but currently requires basic programming skills to be used.
- The ComaCreator plugin is in a long-untested standalone state — it is mandatory to use it through SISPO, and the authors state it should only be used for creating pretty realistic looking pictures, not as a substitute for real science.
- ComaCreator output files (EXR) can grow to 1 GB easily; decimating the shape model to ~3000 triangles (decimate_percentage 0.001) is recommended for performance.
- Docker deployments need a large VM partition (default 10 GB is not enough) and core/memory allocation must be configured via docker-machine before use.

## Evidence

- [abstract] Physically based path-traced rendering via Cycles: "The image simulator utilises the open-source 3-D visualisation system Blender and its Cycles rendering engine, which supports physically based rendering capabilities"
- [intro] Rendering pipeline scope: "The Space Imaging Simulator for Proximity Operations (SISPO) has been developed to provide a full pipeline from simulated imagery to final data products (e.g., 3-D models)"
- [abstract] Trajectory, light and camera simulation: "SISPO allows the simulation of trajectories, light parameters and camera's intrinsic parameters"
- [abstract] Aberration models: "tools to simulate the most common image aberrations, such as tangential and sagittal astigmatism, internal and external comatic aberration, and simple geometric distortions"
- [abstract] Dust/gas environment models: "has supplementary models to produce realistic dust- and gas-environment optical models for comets and active asteroids"
- [discussion] Procedural regolith surface: "For regolith surface reconstruction it uses procedural texturing."
- [discussion] Path-tracing workflow step: [evidence span withheld — anchor status: not_found]
- [intro] Itokawa test model: "The 25143 Itokawa asteroid model by the [14] was used."
- [readme] ComaCreator must run through SISPO: [evidence span withheld — anchor status: external_doc]
- [supplementary] Image comparison script: "S1 ScriptImage comparison.The file contains python algorithm used to compare images in this manuscript."
