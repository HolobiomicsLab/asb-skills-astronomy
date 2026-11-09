---
name: small-body-flyby-scene-rendering
description: Use when you need rapid generation of synthetic spacecraft camera images
  of a small-body (e.g., asteroid 25143 Itokawa) fly-by, or closed-loop GNC/visual-navigation
  simulation where per-image render time must be seconds, not minutes.
when_to_use_negative:
- When photorealistic, physically based images are required for instrument performance
  characterisation — use SISPO's Cycles path-tracing engine instead of the lightweight
  OpenGL renderer
- When the target requires dust/gas coma modelling via ComaCreator, which is only
  supported with Cycles rendering
- When the user has no programming skills — the article notes the tool is open access
  but currently requires basic programming skills to be used
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: SISPO
  role: Primary imaging simulator; provides the CLI, camera model, trajectory simulation
    and the OpenGL lightweight renderer for fly-by image generation
  repo: https://github.com/SISPO-developers/sispo
- name: Blender
  role: Underlying 3-D visualisation package used by SISPO; hosts the Cycles path-tracer
    for the photorealistic alternative path
  repo: https://github.com/SISPO-developers/sispo
- name: sispo_docker
  role: Docker-based deployment that simplifies installation of SISPO and its environment
  repo: https://github.com/SISPO-developers/sispo_docker
- name: ComaCreator
  role: Optional dust/gas environment plugin invoked by SISPO before rendering (Cycles
    support only)
  repo: https://github.com/SISPO-developers/ComaCreator
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pone.0263882
    title: 'SISPO: Space Imaging Simulator for Proximity Operations'
  source_content_hash: sha256:8d63c058fdff80899d31e7c427d5c192d83dd5e1b600bcccb38f9d51d4c97fb0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SISPO-developers/sispo
  tools:
  - Blender
  - ComaCreator
  - SISPO
  - sispo_docker
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# small-body-flyby-scene-rendering

## Summary

Render synthetic images of a small solar system body fly-by scene using SISPO's lightweight OpenGL-based renderer, trading photorealism for speed when many images or closed-loop simulations are needed.

## When to use

Use when you need rapid generation of synthetic spacecraft camera images of a small-body (e.g., asteroid 25143 Itokawa) fly-by, or closed-loop GNC/visual-navigation simulation where per-image render time must be seconds, not minutes.

## When NOT to use

- When photorealistic, physically based images are required for instrument performance characterisation — use SISPO's Cycles path-tracing engine instead of the lightweight OpenGL renderer
- When the target requires dust/gas coma modelling via ComaCreator, which is only supported with Cycles rendering
- When the user has no programming skills — the article notes the tool is open access but currently requires basic programming skills to be used

## Inputs

- 3-D shape model of the target body (e.g., 25143 Itokawa asteroid model)
- SISPO input file defining fly-by trajectory, illumination, camera intrinsic parameters, and renderer choice (OpenGL vs. Cycles)
- Optional ComaCreator plugin configuration for dust/gas environments (Cycles-only rendering)

## Outputs

- Sequence of rendered fly-by images suitable for visual navigation or 3-D reconstruction
- Per-image generation times and total closed-loop mission-week wall-clock time (e.g., tabulated as render_benchmark.csv)

## How to apply

Install sispo as a Python package via pip (or use the sispo_docker image) to obtain its CLI, then load the 3-D shape model of the target — the article used the 25143 Itokawa asteroid model — and configure the SISPO camera model to select the lightweight OpenGL-based renderer instead of Blender's Cycles path-tracer. Define the fly-by trajectory, illumination and camera intrinsic parameters in the SISPO input file, then run the renderer over the image sequence, recording per-image wall-clock time. For closed-loop GNC analysis, feed the rendered frames into the navigation loop over one simulated mission week and record total wall-clock time. Use the Cycles path-tracer only when photorealistic physically based output is required; the OpenGL path is the performance-oriented alternative (≈0.7–1.0 s/image and ≈12 h per simulated mission week on an Intel i7-7700HQ at 2.8 GHz).

## Related tools

- **SISPO** (Primary imaging simulator; provides the CLI, camera model, trajectory simulation and the OpenGL lightweight renderer for fly-by image generation) — https://github.com/SISPO-developers/sispo
- **Blender** (Underlying 3-D visualisation package used by SISPO; hosts the Cycles path-tracer for the photorealistic alternative path) — https://github.com/SISPO-developers/sispo
- **sispo_docker** (Docker-based deployment that simplifies installation of SISPO and its environment) — https://github.com/SISPO-developers/sispo_docker
- **ComaCreator** (Optional dust/gas environment plugin invoked by SISPO before rendering (Cycles support only)) — https://github.com/SISPO-developers/ComaCreator

## Examples

```
pip install sispo; sispo --with-render --with-sim  # render fly-by sequence with lightweight renderer and record per-image times
```

## Evaluation signals

- Per-image render time on i7-7700HQ-class hardware should fall in the ~0.7–1.0 s range for the OpenGL renderer; values an order of magnitude higher suggest Cycles was used by mistake
- One simulated mission week of closed-loop GNC should complete in roughly ~12 hours wall-clock on the same laptop-class hardware
- Rendered image sequence should be consistent with the configured trajectory, illumination and camera intrinsic parameters in the SISPO input file
- Output images should be checkable against the input Itokawa shape model geometry (correct viewing phase and resolution for the fly-by geometry)

## Limitations

- The lightweight OpenGL renderer trades photorealism for speed; physically based rendering requires Cycles, which is far slower
- Tool usage requires basic programming skills, limiting accessibility
- ComaCreator dust/gas environments work only with Cycles rendering and are explicitly not science-grade ('should be only used for creating pretty realistic looking pictures')
- Large meshes benefit from decimation: ComaCreator's default decimates ~3 million triangles down to ~3000 because full resolution 'will take forever to iterate'

## Evidence

- [discussion] OpenGL renderer timing: [evidence span withheld — anchor status: not_found]
- [discussion] Cycles path-tracer: "The tool generates physically based, photorealistic images from an input 3-D model using Blender’s Cycles path-tracing rendering engine."
- [methods] CLI installation: "If sispo is installed as a python package via pip, sispo provides a CLI."
- [intro] Itokawa model: "The 25143 Itokawa asteroid model by the [14] was used."
- [abstract] simulation scope: "SISPO allows the simulation of trajectories, light parameters and camera’s intrinsic parameters"
- [readme] Docker install: [evidence span withheld — anchor status: external_doc]
