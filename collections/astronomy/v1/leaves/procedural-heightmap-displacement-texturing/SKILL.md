---
name: procedural-heightmap-displacement-texturing
description: Use when a plain 3-D asteroid/SSSB shape model (e.g. the 25143 Itokawa
  mesh) must be augmented with realistic procedural regolith, rock and crater detail
  for photorealistic fly-by image simulation.
when_to_use_negative:
- The input is already a photorealistic or instrument-observed textured model that
  must be preserved as-is, since procedural displacement would alter the surface geometry
  on top of the plain input mesh
- The goal is quantitative terrain modelling or scientific shape reconstruction rather
  than qualitative visual realism for image simulation
- Rendering must use an engine other than Cycles (e.g. the ComaCreator plugin path
  is documented as 'Only Cycles support')
tools:
- name: SISPO
  role: Full imaging-simulation pipeline that provides the automatic procedural surface
    generation step feeding the textured mesh into rendering
  repo: https://github.com/SISPO-developers/sispo
- name: Blender
  role: 3-D visualisation system whose Python API is scripted to load the plain mesh
    and build the procedural height-map shader nodes
  repo: https://github.com/SISPO-developers/sispo
- name: Cycles
  role: Physically based path-tracing engine that evaluates the height-field shader
    nodes, applies displacement, and renders the final surface image
  repo: https://github.com/SISPO-developers/sispo
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pone.0263882
    title: 'SISPO: Space Imaging Simulator for Proximity Operations'
  source_content_hash: sha256:2468989e3e164a3b8c82e5a46a0e9816b30bccfde1c2d3ef4b8c61204de419b5
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
yes_rate: 0.6
n_claims: 10
---

# procedural-heightmap-displacement-texturing

## Summary

Reconstructs realistic small-body surface features (sand flats, rock formations, craters) on a plain input asteroid mesh by generating procedural height maps inside Blender's Cycles shader nodes, blending them with per-layer weights and masks, and applying them as displacement with a matching asteroid-albedo texture, as implemented in SISPO.

## When to use

Use when a plain 3-D asteroid/SSSB shape model (e.g. the 25143 Itokawa mesh) must be augmented with realistic procedural regolith, rock and crater detail for photorealistic fly-by image simulation.

## When NOT to use

- The input is already a photorealistic or instrument-observed textured model that must be preserved as-is, since procedural displacement would alter the surface geometry on top of the plain input mesh
- The goal is quantitative terrain modelling or scientific shape reconstruction rather than qualitative visual realism for image simulation
- Rendering must use an engine other than Cycles (e.g. the ComaCreator plugin path is documented as 'Only Cycles support')

## Inputs

- Plain, undistorted 3-D asteroid shape model (e.g. 25143 Itokawa model, .obj mesh)
- Per-layer terrain parameters: height-map weights and mask textures for sand flats, rocks, and craters
- Real asteroid albedo value(s) for the albedo texture

## Outputs

- Displaced, procedurally textured 3-D mesh ready for Cycles rendering
- Rendered image of the textured surface (e.g. procedural_surface_render.png)

## How to apply

Load the plain 3-D mesh of the target small body (e.g. the 25143 Itokawa model) into Blender through its Python scripting API, which SISPO uses because it 'allows for programming procedures in Python'. Implement procedural height-map generators for the three terrain archetypes — sand flats, rock formations, and craters — as Cycles shader-node height fields driven from Python. Blend the height maps with modifiable per-map weights and restrict each to specific mesh areas using masks, and derive a matching albedo texture from the same procedural noise fields using real asteroid albedo values. Enable Cycles displacement on the material so the height-map displacement is applied on top of the undistorted plain input mesh, preserving the base geometry, then render the displaced, textured mesh with Blender's Cycles path-tracing engine. Because no quantitative benchmark is reported for this mechanism in isolation, accept the result by qualitative visual comparison against the paper's demonstration figures (Figs 3, 4, 7, 8).

## Related tools

- **SISPO** (Full imaging-simulation pipeline that provides the automatic procedural surface generation step feeding the textured mesh into rendering) — https://github.com/SISPO-developers/sispo
- **Blender** (3-D visualisation system whose Python API is scripted to load the plain mesh and build the procedural height-map shader nodes) — https://github.com/SISPO-developers/sispo
- **Cycles** (Physically based path-tracing engine that evaluates the height-field shader nodes, applies displacement, and renders the final surface image) — https://github.com/SISPO-developers/sispo

## Examples

```
pip install sispo  # installs the SISPO package, which then provides a CLI for driving the Blender/Cycles procedural surface rendering pipeline
```

## Evaluation signals

- procedural_surface_render.png exists and is a valid, renderable image file
- Render qualitatively matches the paper's demonstration figures (Figs 3, 4, 7, 8) for sand, rock and crater surfaces
- The base 3-D geometry is preserved: displacement is applied on top of the undistorted plain input mesh rather than replacing it
- All three height-map layers (craters, rocks, sand flats) are present, each with a modifiable weight and an area mask
- Albedo of the rendered surface corresponds to real asteroid albedo values, not arbitrary colours

## Limitations

- No quantitative benchmark exists for this mechanism in isolation; validation is only a qualitative visual check
- SISPO is open access but currently requires basic programming skills to be used
- The approach simulates elevation detail procedurally, so surfaces are plausible regolith patterns rather than measured terrain

## Evidence

- [abstract] The image simulator utilises the open-source 3-D visualisation system Blender and its Cycles rendering engine, which supports physically based rendering capabilities: "The image simulator utilises the open-source 3-D visualisation system Blender and its Cycles rendering engine, which supports physically based rendering capabilities"
- [discussion] Procedural texturing for regolith: "For regolith surface reconstruction it uses procedural texturing."
- [discussion] Path-tracing rendering of input 3-D model: "The tool generates physically based, photorealistic images from an input 3-D model using Blender’s Cycles path-tracing rendering engine."
- [intro] Python programmability of Blender: "SISPO uses Cycles rendering with the Blender software package, which allows for programming procedures in Python [15]."
- [intro] Itokawa model as dataset: "The 25143 Itokawa asteroid model by the [14] was used."
