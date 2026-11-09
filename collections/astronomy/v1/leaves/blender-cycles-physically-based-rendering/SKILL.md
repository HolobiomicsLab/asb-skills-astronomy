---
name: blender-cycles-physically-based-rendering
description: Use when synthetic, physically based images of a small solar system body
  (fly-by or orbiting scenarios) or terrestrial planet surface must be rendered from
  a 3-D model to support mission design, instrument characterisation, or computer-vision
  algorithm development such as SfM/MVS 3-D reconstruction.
when_to_use_negative:
- You already hold real acquired imagery of the target — no synthetic rendering is
  needed and reconstruction should run directly on the real images.
- 'ComaCreator dust/gas realism is required for actual science: the README states
  it ''should be only used for creating pretty realistic looking pictures'' and is
  in development stage.'
- The scene requires legacy or proprietary rendering fidelity already covered by dedicated
  tools such as SurRender or PANGU rather than SISPO's open-source Blender/Cycles
  stack.
edam_operation: http://edamontology.org/operation_0337
tools:
- name: SISPO
  role: Pipeline that drives the simulation from input 3-D model to rendered images
    and final data products (e.g., 3-D models); installed as a Python package via
    pip or via sispo_docker
  repo: github.com/SISPO-developers/sispo
- name: Blender
  role: Open-source 3-D visualisation system hosting the rendering; supports Python-programmed
    procedures
- name: Cycles
  role: Blender's path-tracing rendering engine providing the physically based rendering
    of the scene
- name: ComaCreator
  role: Optional plugin generating dust and gas environments for comets/active asteroids;
    invoked before rendering and only supported with Cycles
- name: sispo_docker
  role: Docker image recommended for the easiest installation of SISPO with an environment
    ready for ComaCreator
  repo: https://github.com/SISPO-developers/sispo_docker
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pone.0263882
    title: 'SISPO: Space Imaging Simulator for Proximity Operations'
  source_content_hash: sha256:10c51735bb5be6044d9e6d6054bc9185a6269fed5a1b169db0815db9a1ba33ad
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Blender
  - ComaCreator
  - Cycles
  - OpenMVG
  - OpenMVS
  - SISPO
  - sispo_docker
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# blender-cycles-physically-based-rendering

## Summary

Generates physically based, photorealistic synthetic images from an input 3-D model (e.g., the 25143 Itokawa asteroid shape model) using SISPO's Blender/Cycles path-tracing pipeline, so the rendered image sets can feed downstream computer-vision products such as OpenMVG/OpenMVS 3-D reconstruction.

## When to use

Use when synthetic, physically based images of a small solar system body (fly-by or orbiting scenarios) or terrestrial planet surface must be rendered from a 3-D model to support mission design, instrument characterisation, or computer-vision algorithm development such as SfM/MVS 3-D reconstruction.

## When NOT to use

- You already hold real acquired imagery of the target — no synthetic rendering is needed and reconstruction should run directly on the real images.
- ComaCreator dust/gas realism is required for actual science: the README states it 'should be only used for creating pretty realistic looking pictures' and is in development stage.
- The scene requires legacy or proprietary rendering fidelity already covered by dedicated tools such as SurRender or PANGU rather than SISPO's open-source Blender/Cycles stack.

## Inputs

- Input 3-D shape model (e.g., 25143 Itokawa asteroid model, .obj mesh)
- Trajectory definition (fly-by or orbiting scenario around a spinning target)
- Light source parameters and camera intrinsic parameters (e.g., pinhole camera)
- Optional ComaCreator dust/gas environment configuration (defaults.in / input JSON with plugins and options fields)

## Outputs

- Photorealistic, physically based rendered image set (Cycles path-traced; ComaCreator outputs EXR files for the dust/gas environment)
- Image set suitable as input to OpenMVG structure-from-motion and OpenMVS multi-view stereo for 3-D model reconstruction

## How to apply

Load the input 3-D shape model into SISPO (e.g., the 25143 Itokawa asteroid model), then configure trajectories, light parameters, and camera intrinsic parameters such as a pinhole camera model. Render the image set with Blender's Cycles path-tracing engine, choosing an acquisition geometry appropriate to the study — e.g., 53 images from different angles around a spinning SSSB in an orbiting scenario. Regolith surface detail is reconstructed procedurally via procedural texturing, and optional image aberrations (tangential and sagittal astigmatism, internal and external comatic aberration, geometric distortions) can be simulated. The rendered images then serve as input to structure-from-motion (OpenMVG) and multi-view stereo (OpenMVS); accuracy scales with the number of images and their resolution, so increase these if reconstruction deviation statistics are too coarse.

## Related tools

- **SISPO** (Pipeline that drives the simulation from input 3-D model to rendered images and final data products (e.g., 3-D models); installed as a Python package via pip or via sispo_docker) — github.com/SISPO-developers/sispo
- **Blender** (Open-source 3-D visualisation system hosting the rendering; supports Python-programmed procedures)
- **Cycles** (Blender's path-tracing rendering engine providing the physically based rendering of the scene)
- **ComaCreator** (Optional plugin generating dust and gas environments for comets/active asteroids; invoked before rendering and only supported with Cycles)
- **sispo_docker** (Docker image recommended for the easiest installation of SISPO with an environment ready for ComaCreator) — https://github.com/SISPO-developers/sispo_docker

## Examples

```
"options": ["--with-render","--with-sim", "--with-plugin"]  # sispo input JSON: enable Cycles rendering (and optional ComaCreator plugin) before running sispo
```

## Evaluation signals

- Rendered images must be produced by Cycles path tracing from the input 3-D model (physically based, photorealistic), matching the acquisition geometry configured (e.g., 53 pinhole-camera images from different angles around the spinning target).
- Downstream validation: SfM (OpenMVG) + MVS (OpenMVS) on the rendered set should yield a fully reconstructed 3-D model comparable to the input reference.
- Reproduce the SSSB orbiting case-study statistics: Gaussian mean error ~1.2 m, standard deviation ~6.6 m, with crater-spot deviations in the 60–89.7 m range.
- Model-deviation statistics computed with the supplementary S1 image-comparison Python algorithm should fall within the reported ranges; larger deviations indicate too few images or too low resolution.
- If ComaCreator is enabled, verify the EXR dust/gas environment files initialise Cycles correctly via the final_output JSON.

## Limitations

- SISPO is open access but currently requires basic programming skills to be used.
- ComaCreator's dust/gas rendering only works with Cycles, is in development stage, and per the README should not be used as a substitute for real science — only for visually realistic pictures.
- ComaCreator EXR output files can easily grow to ~1 GB; very large meshes (~3 million triangles) must be decimated (e.g., to ~3000 triangles, decimate_percentage 0.001) or rendering iterates impractically slowly.
- Reconstruction accuracy depends on image count and resolution: errors of 60–89.7 m at crater spots in the 53-image orbiting case can be reduced with more images and higher resolution.
- SISPO contains no changelog and future improvements are deferred to a future-work section.

## Evidence

- [abstract] Cycles supports physically based rendering: "The image simulator utilises the open-source 3-D visualisation system Blender and its Cycles rendering engine, which supports physically based rendering capabilities"
- [abstract] SISPO allows simulating trajectories, light and camera parameters: "SISPO allows the simulation of trajectories, light parameters and camera’s intrinsic parameters"
- [discussion] Cycles path-tracing from input 3-D model: "The tool generates physically based, photorealistic images from an input 3-D model using Blender’s Cycles path-tracing rendering engine."
- [intro] Python programmability of Blender/Cycles in SISPO: "SISPO uses Cycles rendering with the Blender software package, which allows for programming procedures in Python"
- [readme] Enabling rendering (and plugin) via sispo input options: [evidence span withheld — anchor status: external_doc]
- [readme] ComaCreator renders only with Cycles: [evidence span withheld — anchor status: external_doc]
