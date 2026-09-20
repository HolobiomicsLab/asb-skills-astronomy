---
name: small-body-flyby-image-series-simulation
description: Use when you need synthetic, physically based fly-by images of an SSSB
  from an input 3-D shape model with controlled trajectory, light parameters, and
  camera intrinsics (e.g., to feed an SfM/MVS pipeline or test imaging algorithms).
when_to_use_negative:
- The goal is 3-D reconstruction itself (SfM/MVS/CloudCompare error statistics) rather
  than generating the imagery that feeds it — that is a downstream skill operating
  on this skill's output
- 'The imagery must support actual cometary science: ComaCreator''s authors state
  it is in development ''forever'' and should only be used for creating pretty realistic-looking
  pictures, not as a substitute for real science'
- The source data are real spacecraft images rather than an input 3-D model — SISPO
  simulates from a model, it does not process observational imagery
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: SISPO
  role: Full pipeline from simulated imagery to final data products; invoked via CLI
    to render the fly-by image series from the shape model and scenario configuration
  repo: github.com/SISPO-developers/sispo
- name: Blender
  role: 3-D visualisation package driving the rendering; procedures are programmable
    in Python
- name: Cycles
  role: Blender's path-tracing rendering engine used to generate physically based,
    photorealistic images
- name: ComaCreator
  role: Optional SISPO plugin generating dust- and gas-environment models (JetCreator
    invoked automatically before Cycles rendering) for comets and active asteroids
  repo: https://github.com/SISPO-developers/ComaCreator
- name: sispo_docker
  role: Docker image to deploy SISPO quickly with a ready environment (recommended
    way to use SISPO with ComaCreator)
  repo: https://github.com/SISPO-developers/sispo_docker
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pone.0263882
    title: 'SISPO: Space Imaging Simulator for Proximity Operations'
  source_content_hash: sha256:ea81eda4aef2ec45eb3c9f5ba2919b58f138d425c4b70a2b6a98e7ce215074a1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Blender
  - ComaCreator
  - Cycles
  - SISPO
  - sispo_docker
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# small-body-flyby-image-series-simulation

## Summary

Render a physically based, photorealistic fly-by image series of a small solar system body (e.g., the 25143 Itokawa shape model) from a configured trajectory, light, and pinhole-camera setup using SISPO and Blender's Cycles path-tracing engine, for downstream tasks such as SfM 3-D reconstruction or computer-vision algorithm development.

## When to use

Use when you need synthetic, physically based fly-by images of an SSSB from an input 3-D shape model with controlled trajectory, light parameters, and camera intrinsics (e.g., to feed an SfM/MVS pipeline or test imaging algorithms).

## When NOT to use

- The goal is 3-D reconstruction itself (SfM/MVS/CloudCompare error statistics) rather than generating the imagery that feeds it — that is a downstream skill operating on this skill's output
- The imagery must support actual cometary science: ComaCreator's authors state it is in development 'forever' and should only be used for creating pretty realistic-looking pictures, not as a substitute for real science
- The source data are real spacecraft images rather than an input 3-D model — SISPO simulates from a model, it does not process observational imagery

## Inputs

- 3-D shape model of the target small body (e.g., 25143 Itokawa asteroid model; .obj mesh such as comet3.obj for ComaCreator)
- Fly-by scenario configuration: spacecraft trajectory, light parameters, and pinhole camera intrinsic parameters in a SISPO input file (JSON with options such as --with-render, --with-sim)
- Optional ComaCreator settings for dust/gas environments (e.g., n_particles, sample_points grid 64x64x64, bounding box)

## Outputs

- Photorealistic, physically based fly-by image series (e.g., a 25-image set) rendered by Blender's Cycles engine
- EXR-format image frames (ComaCreator converts dust/gas fields to EXR for Cycles initialization)

## How to apply

Install SISPO as a Python package via pip (which provides a CLI) or via the sispo_docker Docker image. Prepare an input 3-D shape model (e.g., the 25143 Itokawa asteroid model) and configure the fly-by scenario: spacecraft trajectory, light parameters, and pinhole camera intrinsic parameters in the SISPO input file. Invoke the renderer with --with-render (and --with-sim) so Blender's Cycles path-tracing engine generates the physically based photorealistic image series; for a comet or active asteroid, enable the ComaCreator plugin (--with-plugin) to overlay dust/gas environments, decimating large meshes (e.g., from ~3 million to ~3000 triangles) before simulation. The output is an image series (EXR frames) that can then be compared (SISPO's image-comparison script) or passed to structure-from-motion. Expect the fly-by case used in the paper to be a 25-image series rendered with Cycles as the input to OpenMVG/OpenMVS reconstruction.

## Related tools

- **SISPO** (Full pipeline from simulated imagery to final data products; invoked via CLI to render the fly-by image series from the shape model and scenario configuration) — github.com/SISPO-developers/sispo
- **Blender** (3-D visualisation package driving the rendering; procedures are programmable in Python)
- **Cycles** (Blender's path-tracing rendering engine used to generate physically based, photorealistic images)
- **ComaCreator** (Optional SISPO plugin generating dust- and gas-environment models (JetCreator invoked automatically before Cycles rendering) for comets and active asteroids) — https://github.com/SISPO-developers/ComaCreator
- **sispo_docker** (Docker image to deploy SISPO quickly with a ready environment (recommended way to use SISPO with ComaCreator)) — https://github.com/SISPO-developers/sispo_docker

## Examples

```
pip install sispo; sispo --with-render --with-sim --input flyby_config.json
```

## Evaluation signals

- Rendered frames are physically based path-traced outputs of Cycles, not rasterized approximations
- The image series matches the configured scenario (e.g., 25 images for the fly-by case) with the specified pinhole camera intrinsics, trajectory, and lighting
- Images are in the expected format (EXR when ComaCreator's dust/gas field is converted for Cycles)
- Downstream sanity check: the rendered 25-image SSSB fly-by series should support SfM reconstruction with reported Gaussian mean error of 14.7 m (std 63.9 m) against the input Itokawa model
- Image-comparison script (S1 ScriptImage comparison) can be run to quantify differences between rendered images

## Limitations

- SISPO is open access but currently requires basic programming skills to be used
- ComaCreator is in a permanent development stage, has bugs, and should only be used for creating pretty realistic-looking pictures, not real science
- EXR outputs grow easily to ~1 GB per file; large shape meshes (~3 million triangles) must be decimated (e.g., to 3000 triangles) before ComaCreator simulation
- sispo_docker requires a Docker-capable machine; the default VirtualBox VM partition (10 GB) is not enough and must be enlarged

## Evidence

- [abstract] Physically based path-traced rendering with Cycles: "The image simulator utilises the open-source 3-D visualisation system Blender and its Cycles rendering engine, which supports physically based rendering capabilities"
- [abstract] Scenario parameters are controllable: "SISPO allows the simulation of trajectories, light parameters and camera's intrinsic parameters"
- [intro] Full pipeline intent: "The Space Imaging Simulator for Proximity Operations (SISPO) has been developed to provide a full pipeline from simulated imagery to final data products (e.g., 3-D models)"
- [intro] Input shape model: "The 25143 Itokawa asteroid model by the [14] was used."
- [methods] CLI installation: "If sispo is installed as a python package via pip, sispo provides a CLI."
- [discussion] Rendering step and engine: [evidence span withheld — anchor status: not_found]
- [readme] Plugin invocation options: [evidence span withheld — anchor status: external_doc]
- [readme] EXR output format: [evidence span withheld — anchor status: external_doc]
