---
name: noise-mask-based-texture-blending
description: Use when a plain 3-D asteroid/small-body mesh (e.g., the 25143 Itokawa
  model) must gain realistic surface elevation and albedo detail in Blender/Cycles
  while preserving the undistorted input geometry.
when_to_use_negative:
- The input mesh already carries measured elevation/displacement detail (e.g., a real
  shape model with high-resolution topography) — procedural height maps would add
  synthetic detail on top of real data
- 'The goal is quantitative surface-geometry ground truth: the paper reports no quantitative
  benchmark for this mechanism, only a qualitative visual check'
- Rendering is being done outside Blender/Cycles (e.g., purely geometric processing
  of the shape model), since the skill relies on Cycles shader-node displacement and
  path tracing
tools:
- name: SISPO
  role: Pipeline that performs automatic procedural surface generation (regolith surface
    reconstruction) on the input asteroid mesh
  repo: https://github.com/SISPO-developers/sispo
- name: Blender
  role: Hosts the shader-node graph; the mesh and procedural height-map/material nodes
    are driven through its Python scripting API
- name: Cycles
  role: Path-tracing engine that applies the height-map displacement and renders the
    textured, displaced mesh
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pone.0263882
    title: 'SISPO: Space Imaging Simulator for Proximity Operations'
  source_content_hash: sha256:4d56a621248fd9492ebd64b9520912639b6d9014268be11602cad018b144b56b
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
n_claims: 5
---

# noise-mask-based-texture-blending

## Summary

Adds realistic regolith detail (sand flats, rock formations, craters) to a plain small-body mesh by layering procedural height maps with per-map weights and masks inside Blender/Cycles, plus a matching asteroid-albedo texture, without altering the base geometry.

## When to use

Use when a plain 3-D asteroid/small-body mesh (e.g., the 25143 Itokawa model) must gain realistic surface elevation and albedo detail in Blender/Cycles while preserving the undistorted input geometry.

## When NOT to use

- The input mesh already carries measured elevation/displacement detail (e.g., a real shape model with high-resolution topography) — procedural height maps would add synthetic detail on top of real data
- The goal is quantitative surface-geometry ground truth: the paper reports no quantitative benchmark for this mechanism, only a qualitative visual check
- Rendering is being done outside Blender/Cycles (e.g., purely geometric processing of the shape model), since the skill relies on Cycles shader-node displacement and path tracing

## Inputs

- Plain (undistorted) 3-D asteroid mesh, e.g., the 25143 Itokawa asteroid model loaded via Blender's Python API
- Procedural height-map generator definitions for sand flats, rock formations, and craters (Cycles shader-node height fields driven from Python)
- Per-map weight values and mask textures restricting each height map to specific mesh areas
- Asteroid albedo texture derived from the same procedural noise fields

## Outputs

- Displaced, procedurally textured mesh material with layered height-map displacement and asteroid albedo
- Rendered image of the displaced, textured mesh (procedural_surface_render.png) from the Cycles path-tracing engine

## How to apply

Load the plain 3-D mesh (e.g., the 25143 Itokawa model) into Blender via its Python scripting API and build Cycles shader-node height fields for the three terrain archetypes: sand flats, rock formations, and craters. Blend the three height maps using modifiable per-map weights, and restrict each map to specific mesh areas using masks so features appear only where appropriate. Derive an albedo texture corresponding to real asteroid albedo from the same procedural noise fields, so elevation and reflectance stay consistent. Enable Cycles displacement on the material so the height-map displacement is applied on top of the undistorted plain input mesh, preserving the base geometry. Render with Cycles path tracing; validation is qualitative — the render (procedural_surface_render.png) should match the paper's demonstration figures (Figs 3, 4, 7, 8), since the paper reports no quantitative benchmark for this mechanism in isolation.

## Related tools

- **SISPO** (Pipeline that performs automatic procedural surface generation (regolith surface reconstruction) on the input asteroid mesh) — https://github.com/SISPO-developers/sispo
- **Blender** (Hosts the shader-node graph; the mesh and procedural height-map/material nodes are driven through its Python scripting API)
- **Cycles** (Path-tracing engine that applies the height-map displacement and renders the textured, displaced mesh)

## Evaluation signals

- procedural_surface_render.png exists and is a valid, renderable image file
- Render qualitatively matches the paper's demonstration figures (Figs 3, 4, 7, 8) in surface character (sand flats, rocks, craters)
- Base mesh geometry is unchanged: displacement is applied on top of the undistorted plain input mesh, not baked into it
- Each terrain archetype appears only within its mask region; per-map weights alter feature intensity predictably
- Albedo texture and elevation derive from the same procedural noise fields, so bright/dark areas correspond consistently to the height field

## Limitations

- No quantitative benchmark exists for this mechanism in isolation; acceptance is a qualitative visual check
- Using SISPO currently requires basic programming skills (Python) to script Blender/Cycles nodes and meshes
- Procedural detail is synthetic — realistic-looking but not necessarily matching the target body's actual surface; albedo is only 'corresponding to real asteroid albedo'

## Evidence

- [discussion] Procedural texturing via layered height maps: "For regolith surface reconstruction it uses procedural texturing."
- [discussion] Rendering engine and displacement context: "The tool generates physically based, photorealistic images from an input 3-D model using Blender’s Cycles path-tracing rendering engine."
- [intro] Python-driven Blender/Cycles setup: "SISPO uses Cycles rendering with the Blender software package, which allows for programming procedures in Python"
- [full_text] Height-map weights and masks finding: [evidence span withheld — anchor status: not_found]
