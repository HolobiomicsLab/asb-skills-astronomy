---
name: density-volume-import-into-blender
description: Use when a simulated gas/dust coma number-density volume (e.g., ComaCreator
  EXR output on a 64x64x64 sample grid) must be visualised as a rendered image of
  a comet nucleus embedded in a diffuse coma within a fly-by scene.
when_to_use_negative:
- The coma density map has not yet been generated — ComaCreator's standalone version
  is untested and 'it is mandatory to use Sispo', so the volume must come from the
  Sispo-invoked JetCreator pipeline first
- The goal is quantitative coma science rather than visualization — ComaCreator 'should
  be only used for creating pretty realistic looking pictures' and is not a substitute
  for real science
- The scene contains no volumetric medium (e.g., a bare asteroid or planetary surface
  fly-by) — a surface shader on the nucleus mesh, not a volume shader, is appropriate
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: SISPO
  role: 'Orchestrates the pipeline: invokes ComaCreator/JetCreator before rendering,
    loads the density volume, and drives the Cycles render'
  repo: https://github.com/SISPO-developers/sispo
- name: Blender
  role: Hosts the scene in which the number-density volume is imported as a volumetric
    density field and shaders are assigned
  repo: https://github.com/SISPO-developers/sispo
- name: Cycles
  role: Path-tracing engine that renders the volumetric scattering or emission shader
    on the coma volume
  repo: https://github.com/SISPO-developers/sispo
- name: ComaCreator
  role: Dust and gas environment generator producing the smoothed density volume (EXR)
    that is imported into Blender
  repo: SISPO-developers/ComaCreator
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1371/journal.pone.0263882
    title: 'SISPO: Space Imaging Simulator for Proximity Operations'
  source_content_hash: sha256:1dface5dea54b9a0881bdef7374ff4f2b257f193c275398b5665619b0be45769
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SISPO-developers/sispo
  tools:
  - Blender
  - ComaCreator
  - Cycles
  - SISPO
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# Density volume import into Blender

## Summary

Loads a tricubic-smoothed 3-D number-density map of a comet coma (produced by ComaCreator/JetCreator) into Blender as a volumetric density field and renders it with Cycles using either a volumetric scattering shader or a faster volumetric emission variant.

## When to use

Use when a simulated gas/dust coma number-density volume (e.g., ComaCreator EXR output on a 64x64x64 sample grid) must be visualised as a rendered image of a comet nucleus embedded in a diffuse coma within a fly-by scene.

## When NOT to use

- The coma density map has not yet been generated — ComaCreator's standalone version is untested and 'it is mandatory to use Sispo', so the volume must come from the Sispo-invoked JetCreator pipeline first
- The goal is quantitative coma science rather than visualization — ComaCreator 'should be only used for creating pretty realistic looking pictures' and is not a substitute for real science
- The scene contains no volumetric medium (e.g., a bare asteroid or planetary surface fly-by) — a surface shader on the nucleus mesh, not a volume shader, is appropriate

## Inputs

- Comet nucleus shape model (e.g., comet3.obj, optionally decimated/triangulated via ComaCreator's decimate_mesh option)
- Tricubic-smoothed 3-D gas number-density volume (EXR output from ComaCreator's ToBlender step, sampled on e.g. a 64x64x64 grid within a bounding box such as [-5,-5,-5] to [5,5,5])
- Sispo input JSON with options ["--with-render","--with-sim","--with-plugin"], plugins list, and ComaCreator parameters (T_gas, m_gas, outflow_rates, n_particles)
- Fly-by scene configuration: illumination, camera parameters, and comet nucleus geometry from the SISPO scene definition

## Outputs

- Rendered image of the comet nucleus embedded in a diffuse coma (e.g., comet_coma_render.png) produced by the Cycles path-tracing engine
- Optional dust particle field rendered alongside the gas density volume (n_particles, e.g., 10000, from KParticleSimulation)

## How to apply

First generate the coma density volume with ComaCreator's pipeline (NSDSGenerator -> KGasSimulation with sample_points=[64,64,64] over a cubic bounding box, e.g. [-5,-5,-5] to [5,5,5], then KParticleSimulation and ToBlender, which writes EXR output used to initialise Cycles). Activate the plugin in the Sispo input file with the --with-plugin option and the plugins key pointing to /app/ComaCreator/interface.py; Sispo invokes JetCreator automatically before rendering, and only Cycles rendering is supported. Load the smoothed number-density volume into the Blender scene as a volumetric density field, map the density values to the shader's density input, and assign either the physically based volume-scattering shader or the faster volumetric emission shader when scattering fidelity is not required. Set up the comet nucleus geometry, illumination, and camera parameters from the SISPO-defined scene configuration, then render with Cycles path tracing. Decimate the nucleus shape model (e.g., decimate_percentage around 0.001, reducing ~3 million triangles to ~3000) because iterating the full mesh takes too long.

## Related tools

- **SISPO** (Orchestrates the pipeline: invokes ComaCreator/JetCreator before rendering, loads the density volume, and drives the Cycles render) — https://github.com/SISPO-developers/sispo
- **Blender** (Hosts the scene in which the number-density volume is imported as a volumetric density field and shaders are assigned) — https://github.com/SISPO-developers/sispo
- **Cycles** (Path-tracing engine that renders the volumetric scattering or emission shader on the coma volume) — https://github.com/SISPO-developers/sispo
- **ComaCreator** (Dust and gas environment generator producing the smoothed density volume (EXR) that is imported into Blender) — SISPO-developers/ComaCreator

## Examples

```
sispo --with-render --with-sim --with-plugin --plugins /app/ComaCreator/interface.py
```

## Evaluation signals

- Output file comet_coma_render.png exists and is a valid image file
- The rendered image qualitatively shows a comet nucleus embedded in a diffuse coma, comparable in appearance to Fig 9 of the manuscript
- The volume shader's density input is driven by the loaded number-density values (not a constant), so coma brightness varies spatially in the render
- Only the Cycles engine is used — ComaCreator rendering support is Cycles-only ('Only Cycles support')
- If using the emission variant, verify the render completes faster than the scattering variant but with reduced physical accuracy

## Limitations

- The volumetric emission variant is faster but less accurate than the volumetric scattering shader
- EXR density files can grow to ~1 GB easily; resolution and file size are open performance issues (OpenVDB suggested as a future improvement)
- ComaCreator 'doesn't function as [Kramer et al.] intended, has more bugs certainly', and should only be used for realistic-looking pictures, not real science
- The standalone ComaCreator version has not been tested in a long time — it is mandatory to use Sispo
- Decimating the nucleus mesh (e.g., to ~3000 triangles) trades geometric fidelity for tractable render time

## Evidence

- [full_text] Volume shader choice: [evidence span withheld — anchor status: not_found]
- [abstract] Physically based rendering: "The image simulator utilises the open-source 3-D visualisation system Blender and its Cycles rendering engine, which supports physically based rendering capabilities"
- [readme] Plugin activation: [evidence span withheld — anchor status: external_doc]
- [readme] Cycles-only support: [evidence span withheld — anchor status: external_doc]
- [readme] Sampling grid and bounding box: [evidence span withheld — anchor status: external_doc]
- [readme] Decimation rationale: [evidence span withheld — anchor status: external_doc]
- [readme] Not for real science: [evidence span withheld — anchor status: external_doc]
