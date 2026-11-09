---
name: space-scene-illumination-and-camera-configuration
description: Use when you need to define lighting conditions, light parameters, and
  camera intrinsic parameters for a synthetic space-scene render — e.g. simulating
  fly-by imaging geometry of an asteroid or comet with SISPO/Blender before invoking
  the Cycles path-tracing render.
when_to_use_negative:
- You already have real mission imagery — SISPO is for synthetic simulation, not for
  processing acquired images.
- You need quantitative, science-grade coma gas/dust densities — the ComaCreator is
  explicitly 'should be only used for creating pretty realistic looking pictures'
  and is not a substitute for real science.
- You need interactive SPICE-based mission visualisation only (e.g. Cosmographia)
  rather than photorealistic path-traced rendering.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: SISPO
  role: Defines the fly-by scene configuration (trajectories, light parameters, camera
    intrinsics) and drives the render pipeline
  repo: https://github.com/SISPO-developers/sispo
- name: Blender
  role: Hosts the scene setup (nucleus geometry, illumination, camera, volume shaders)
    via Python scripting
  repo: https://github.com/SISPO-developers/sispo
- name: Cycles
  role: Path-tracing rendering engine that produces the physically based photorealistic
    images from the configured scene
- name: sispo_docker
  role: Docker deployment providing a ready environment for running SISPO and the
    ComaCreator plugin
  repo: https://github.com/SISPO-developers/sispo_docker
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pone.0263882
    title: 'SISPO: Space Imaging Simulator for Proximity Operations'
  source_content_hash: sha256:fae96321b2b7e25e59d93969de5af9e16817967d1dcffd04b55431a6cbaba649
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SISPO-developers/sispo
  tools:
  - Blender
  - Cycles
  - SISPO
  - sispo_docker
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# space-scene-illumination-and-camera-configuration

## Summary

Configure the fly-by scene illumination (sun light parameters) and camera intrinsic parameters in SISPO's Blender/Cycles scene definition so that path-traced renders of a small-body scene (nucleus, regolith surface, dust/gas coma volume) are photorealistic and physically based.

## When to use

Use when you need to define lighting conditions, light parameters, and camera intrinsic parameters for a synthetic space-scene render — e.g. simulating fly-by imaging geometry of an asteroid or comet with SISPO/Blender before invoking the Cycles path-tracing render.

## When NOT to use

- You already have real mission imagery — SISPO is for synthetic simulation, not for processing acquired images.
- You need quantitative, science-grade coma gas/dust densities — the ComaCreator is explicitly 'should be only used for creating pretty realistic looking pictures' and is not a substitute for real science.
- You need interactive SPICE-based mission visualisation only (e.g. Cosmographia) rather than photorealistic path-traced rendering.

## Inputs

- Input 3-D model of the small body (e.g. OBJ shape model like comet3.obj or the 25143 Itokawa asteroid model)
- SISPO scene configuration (JSON input file) with 'options' such as --with-render / --with-sim, specifying trajectories, light parameters and camera intrinsic parameters
- Optional volumetric coma density field (tricubic-smoothed number density, EXR output from ComaCreator) to include in the scene

## Outputs

- Physically based, photorealistic rendered images of the fly-by scene (e.g. comet_coma_render.png showing a nucleus embedded in a diffuse coma)
- Image sequences suitable for downstream 3-D reconstruction / computer-vision processing

## How to apply

Build the scene in Blender as scripted from SISPO's scene configuration, which simulates trajectories, light parameters and camera intrinsic parameters; place the sun/illumination source and set the camera geometry according to the fly-by trajectory being simulated. Assign the input 3-D model (e.g. the 25143 Itokawa asteroid model or a comet nucleus mesh such as comet3.obj) as scene geometry, and if a coma volume is present load the tricubic-smoothed number-density field and map it to the volumetric shader density. Then render with Blender's Cycles path-tracing engine, which produces physically based, photorealistic images from the configured scene; Cycles is the only render engine supported when the ComaCreator plugin generates the dust/gas environment. SISPO is driven programmatically in Python (installed via pip as a CLI, or via the sispo_docker image), passing options such as --with-render and --with-sim in the input file.

## Related tools

- **SISPO** (Defines the fly-by scene configuration (trajectories, light parameters, camera intrinsics) and drives the render pipeline) — https://github.com/SISPO-developers/sispo
- **Blender** (Hosts the scene setup (nucleus geometry, illumination, camera, volume shaders) via Python scripting) — https://github.com/SISPO-developers/sispo
- **Cycles** (Path-tracing rendering engine that produces the physically based photorealistic images from the configured scene)
- **sispo_docker** (Docker deployment providing a ready environment for running SISPO and the ComaCreator plugin) — https://github.com/SISPO-developers/sispo_docker

## Examples

```
sispo --with-render --with-sim
```

## Evaluation signals

- The rendered output image (e.g. comet_coma_render.png) exists and is a valid image file
- The image qualitatively shows a comet nucleus embedded in a diffuse coma, comparable to Fig 9 of the manuscript
- Illumination direction and camera intrinsics in the render match the trajectory/light parameters specified in the SISPO scene configuration
- When ComaCreator is used, the render was produced with Cycles (the only supported engine) after the plugin ran pre-render

## Limitations

- SISPO is open access but currently requires basic programming skills to be used
- Rendering with volumetric emission is faster but less accurate than volumetric scattering
- Docker deployments: the default VM partition (10 GB) is not enough and core/memory allocation must be raised for the virtual machine

## Evidence

- [abstract] SISPO allows the simulation of trajectories, light parameters and camera's intrinsic parameters: "SISPO allows the simulation of trajectories, light parameters and camera’s intrinsic parameters"
- [discussion] The tool generates physically based, photorealistic images from an input 3-D model via path-tracing rendering: "The tool generates physically based, photorealistic images from an input 3-D model using Blender’s Cycles path-tracing rendering engine."
- [intro] SISPO uses Cycles rendering with Blender, scriptable in Python: "SISPO uses Cycles rendering with the Blender software package, which allows for programming procedures in Python [15]."
- [readme] Sispo input file must declare options including --with-render and --with-sim: [evidence span withheld — anchor status: external_doc]
