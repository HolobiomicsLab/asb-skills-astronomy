---
name: particle-transport-simulation
description: Use when simulating the gas/dust environment (coma) of a comet or active
  asteroid from a mesh shape model, to produce a volumetric density field for physically
  based volume rendering in SISPO/Cycles.
when_to_use_negative:
- When a scientifically validated coma density distribution is required — the ComaCreator
  README warns 'please don't use ComaCreator as substitute for a real science'; it
  is for 'pretty realistic looking pictures' only
- When the target is an inactive body (e.g. the 25143 Itokawa asteroid model in the
  SISPO Case 1 pipeline) with no gas/dust emission to propagate
- When output must be a per-voxel dataset with varied (non-cubic) bounding-box shapes
  — ComaCreator restricts the bounding box to cubes ([-x,-x,-x], [x,x,x])
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: ComaCreator
  role: Standalone dust-and-gas environment generator plugin for SISPO that runs the
    NSDSGenerator -> KGasSimulation -> KParticleSimulation -> ToBlender pipeline and
    writes the OpenEXR volume
  repo: https://github.com/SISPO-developers/ComaCreator
- name: SISPO
  role: Host imaging simulator that invokes the ComaCreator plugin before rendering
    and feeds the OpenEXR density volume to Blender/Cycles
  repo: https://github.com/SISPO-developers/sispo
- name: Blender/Cycles
  role: Volume renderer that consumes the OpenEXR number-density map to render the
    coma photorealistically (Cycles only)
- name: sispo_docker
  role: Docker environment with SISPO and ComaCreator pre-installed, recommended as
    the easiest way to run the pipeline
  repo: https://github.com/SISPO-developers/sispo_docker
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pone.0263882
    title: 'SISPO: Space Imaging Simulator for Proximity Operations'
  source_content_hash: sha256:7ba40969161d67c36ac98110748d6522ef4a2876ca9a275e1cb5df3d2f0421b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SISPO-developers/ComaCreator
  tools:
  - Blender/Cycles
  - ComaCreator
  - SISPO
  - sispo_docker
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# particle-transport-simulation

## Summary

Generate a renderable volumetric coma around a comet nucleus by propagating gas and dust emitted from mesh surface sources through gas-field and particle-simulation stages, then binning particle positions into a 3-D number-density map encoded as a single OpenEXR file for Blender/Cycles volume rendering.

## When to use

Use when simulating the gas/dust environment (coma) of a comet or active asteroid from a mesh shape model, to produce a volumetric density field for physically based volume rendering in SISPO/Cycles.

## When NOT to use

- When a scientifically validated coma density distribution is required — the ComaCreator README warns 'please don't use ComaCreator as substitute for a real science'; it is for 'pretty realistic looking pictures' only
- When the target is an inactive body (e.g. the 25143 Itokawa asteroid model in the SISPO Case 1 pipeline) with no gas/dust emission to propagate
- When output must be a per-voxel dataset with varied (non-cubic) bounding-box shapes — ComaCreator restricts the bounding box to cubes ([-x,-x,-x], [x,x,x])

## Inputs

- Comet nucleus shape model mesh (e.g. comet3.obj), optionally decimated/triangulated by ComaCreator (decimate_percentage 0-1, e.g. 0.001 to reduce ~3M triangles to ~3000)
- ComaCreator input file (defaults.in in $ROOT/data/) with GasCreator/programID configuration, sim_common gas parameters (T_gas, m_gas, R_gas), voxel grid settings (sample_points, bbox_lower, bbox_upper) and n_particles
- Gas field output from KGasSimulation, consumed as prevOUTPUT by KParticleSimulation

## Outputs

- 3-D number-density map of gas and dust around the nucleus on a voxel grid, stored as a single OpenEXR file for Blender/Cycles volume rendering
- final_output (e.g. data/out.json) used to initialize Cycles in SISPO

## How to apply

Assign a gas source strength and outflow velocity to every face of the input nucleus mesh (e.g. comet3.obj, decimated to ~3000 triangles with decimate_percentage 0.001) by sampling a noise function, using parameters such as T_gas=210.0, m_gas=0.01801528, and R_gas=0.00046152. Propagate the gas with the KGasSimulation program, sampling the coma on a 64x64x64 voxel grid inside a cubic bounding box such as bbox_lower [-5,-5,-5] to bbox_upper [5,5,5]. Advance the dust ensemble with KParticleSimulation (e.g. n_particles=10000), consuming the gas field output from the previous stage ('prevOUTPUT'). Convert particle positions via ToBlender into a single OpenEXR file, which is the volumetric format Cycles consumes, and smooth the gridded density with tricubic interpolation to remove voxel artifacts. Chain the stages in the pipeline order DefaultGasPreprocessor -> DefaultGasDensityCreator -> DefaultParticleGenerator -> DefaultToBlender, activating ComaCreator as a SISPO plugin with the --with-plugin option and validating only qualitatively, since the paper presents the chain as a proof of concept without isolated quantitative validation.

## Related tools

- **ComaCreator** (Standalone dust-and-gas environment generator plugin for SISPO that runs the NSDSGenerator -> KGasSimulation -> KParticleSimulation -> ToBlender pipeline and writes the OpenEXR volume) — https://github.com/SISPO-developers/ComaCreator
- **SISPO** (Host imaging simulator that invokes the ComaCreator plugin before rendering and feeds the OpenEXR density volume to Blender/Cycles) — https://github.com/SISPO-developers/sispo
- **Blender/Cycles** (Volume renderer that consumes the OpenEXR number-density map to render the coma photorealistically (Cycles only))
- **sispo_docker** (Docker environment with SISPO and ComaCreator pre-installed, recommended as the easiest way to run the pipeline) — https://github.com/SISPO-developers/sispo_docker

## Examples

```
"options": ["--with-render","--with-sim", "--with-plugin"], "plugins": ["/app/ComaCreator/interface.py"]
```

## Evaluation signals

- A single OpenEXR file is produced (ToBlender stage with write_output_to 'final') and file size is expected to be large — the README notes 'We use EXR and the file size can grow to 1 gb easy'
- The density volume is smooth and free of voxel artifacts after tricubic interpolation; the qualitative acceptance check is 'a smooth, physically plausible number-density volume'
- Pipeline executes the four programs in order: DefaultGasPreprocessor, DefaultGasDensityCreator, DefaultParticleGenerator, DefaultToBlender, each consuming prevOUTPUT from the previous stage
- Voxel grid resolution equals sample_points (e.g. 64x64x64) and the volume lies inside the specified cubic bounding box (e.g. -5 to 5)
- SISPO successfully initializes Cycles from the final_output JSON (data/out.json) and renders the volumetric coma

## Limitations

- Proof of concept only: the paper presents the chain without isolated quantitative validation
- ComaCreator has 'more bugs certainly', no collaboration with the original authors, is 'in development stage maybe forever', and should not substitute for real science
- Standalone ComaCreator is untested for a long time — SISPO use is mandatory; rendering supports Cycles only
- Performance: iterating ~3M triangles 'will take forever', requiring mesh decimation; file size can reach ~1 GB; nProcesses must be tuned to available CPU cores

## Evidence

- [full_text] particle position data output from the particle simulation following [42] is further processed into a three-dimensional number-density map around the comet, which is encoded to a single OpenEXR file for subsequent rendering: [evidence span withheld — anchor status: not_found]
- [readme] Dust and gas environment generator for https://github.com/SISPO-developers/sispo.: [evidence span withheld — anchor status: external_doc]
- [readme] This is the resolution of the space around the comet. bbox_lower, and bbox_upper are the bounding box limits. Use cubes ([-x,-x,-x], [x,x,x]): [evidence span withheld — anchor status: external_doc]
- [readme] ComaCreator is in development stage maybe forever and should be only used for creating pretty realistic looking pictures: [evidence span withheld — anchor status: external_doc]
- [full_text] the paper presents this chain only as a proof of concept without isolated quantitative validation: [evidence span withheld — anchor status: not_found]
