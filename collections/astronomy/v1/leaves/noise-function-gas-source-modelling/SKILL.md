---
name: noise-function-gas-source-modelling
description: 'Use when building a cometary dust/gas environment in SISPO Case 2: you
  have a triangulated nucleus mesh (e.g. .obj) and need surface-resolved gas outflow
  rates to initialize the gas-density and particle pipelines before Cycles volume
  rendering.'
when_to_use_negative:
- When quantitative, scientifically validated coma gas/dust densities are required
  — the README states ComaCreator 'should be only used for creating pretty realistic
  looking pictures' and must not substitute for real science.
- 'When the mesh cannot or should not be decimated: the noise-source step depends
  on decimated/triangulated faces (~3000 triangles) and will be extremely slow on
  3-million-triangle models.'
- When running ComaCreator standalone — the README says the standalone version 'has
  not been tested in a long time so it is mandatory to use Sispo'.
edam_operation: http://edamontology.org/operation_0242
tools:
- name: ComaCreator
  role: Implements the noise-function surface source assignment (NSDSGenerator / DefaultGasPreprocessor)
    plus the downstream gas-density, particle, and ToBlender (EXR) stages as a Sispo
    plugin.
  repo: SISPO-developers/ComaCreator
- name: SISPO
  role: Host pipeline that invokes the ComaCreator plugin via --with-plugin and --with-sim
    options before rendering the resulting OpenEXR volume with Cycles.
  repo: github.com/SISPO-developers/sispo
- name: sispo_docker
  role: Recommended deployment with the environment ready for ComaCreator; the easiest
    way to run the gas-source pipeline.
  repo: https://github.com/SISPO-developers/sispo_docker
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1371/journal.pone.0263882
    title: 'SISPO: Space Imaging Simulator for Proximity Operations'
  source_content_hash: sha256:9984ddcd53e5b38bedb61bd9687f0c43396d3f077aa75df28bf5499b4c8cda14
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SISPO-developers/ComaCreator
  tools:
  - ComaCreator
  - SISPO
  - sispo_docker
  edam_operation: http://edamontology.org/operation_0242
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# noise-function-gas-source-modelling

## Summary

Assigns gas source strength and outflow velocity to every face of a comet nucleus shape model by sampling a noise function over the surface, producing the per-face outflow rates that drive SISPO/ComaCreator's gas-field and particle simulations for a renderable coma.

## When to use

Use when building a cometary dust/gas environment in SISPO Case 2: you have a triangulated nucleus mesh (e.g. .obj) and need surface-resolved gas outflow rates to initialize the gas-density and particle pipelines before Cycles volume rendering.

## When NOT to use

- When quantitative, scientifically validated coma gas/dust densities are required — the README states ComaCreator 'should be only used for creating pretty realistic looking pictures' and must not substitute for real science.
- When the mesh cannot or should not be decimated: the noise-source step depends on decimated/triangulated faces (~3000 triangles) and will be extremely slow on 3-million-triangle models.
- When running ComaCreator standalone — the README says the standalone version 'has not been tested in a long time so it is mandatory to use Sispo'.

## Inputs

- Triangulated comet nucleus shape model (e.g. comet3.obj)
- ComaCreator input file (defaults.in) with pipeline order, sim_common gas constants (T_gas, m_gas, R_gas), sample_points and bbox settings
- decimate_mesh/decimate_percentage settings for the shape model

## Outputs

- Per-face gas source strength and outflow velocity table (outflow_rates, passed as prevOUTPUT)
- 3-D gas number-density field on the voxel grid (64x64x64 around [-5,5])
- Single OpenEXR volumetric density file consumed by Blender/Cycles

## How to apply

Decimate and triangulate the input shape model with ComaCreator (decimate_percentage around 0.001 to reduce ~3M triangles to ~3000, since iterating millions of faces is prohibitively slow). The DefaultGasPreprocessor (programID NSDSGenerator) then samples a noise function over the mesh surface to assign each face a gas source strength and outflow velocity, emitting outflow_rates as prevOUTPUT. Pass these rates into DefaultGasDensityCreator (programID KGasSimulation), which propagates the gas with the cited gas-field model using gas constants T_gas=210.0, m_gas=0.01801528, R_gas=0.00046152, on a 64x64x64 sample grid inside a cubic bbox of [-5,5]. The resulting gas field seeds DefaultParticleGenerator (n_particles e.g. 10000), whose particle positions are binned and smoothed (tricubic) by DefaultToBlender into a single OpenEXR volume for Cycles. The noise sampling deliberately produces qualitative, plausible-looking coma structure rather than validated physical fluxes.

## Related tools

- **ComaCreator** (Implements the noise-function surface source assignment (NSDSGenerator / DefaultGasPreprocessor) plus the downstream gas-density, particle, and ToBlender (EXR) stages as a Sispo plugin.) — SISPO-developers/ComaCreator
- **SISPO** (Host pipeline that invokes the ComaCreator plugin via --with-plugin and --with-sim options before rendering the resulting OpenEXR volume with Cycles.) — github.com/SISPO-developers/sispo
- **sispo_docker** (Recommended deployment with the environment ready for ComaCreator; the easiest way to run the gas-source pipeline.) — https://github.com/SISPO-developers/sispo_docker

## Examples

```
{"options": ["--with-render","--with-sim", "--with-plugin"], "plugins": ["/app/ComaCreator/interface.py"]}
```

## Evaluation signals

- outflow_rates from NSDSGenerator are successfully forwarded as prevOUTPUT into KGasSimulation (pipeline order DefaultGasPreprocessor -> DefaultGasDensityCreator -> DefaultParticleGenerator -> DefaultToBlender completes without error).
- The gas-density volume is produced at the requested sample_points resolution (64x64x64) within the cubic bbox [-5,5] on each axis.
- The final output is a single OpenEXR file (final_output path, e.g. data/out.json points to the Cycles initialization) that Cycles can load — only Cycles support.
- Visual check: the rendered coma shows a smooth, physically plausible structure (tricubic-smoothed, no voxel artifacts), not quantitatively validated.
- The EXR file size stays manageable — the README notes sizes 'can grow to 1 gb easy', so watch disk space at high sample_points.

## Limitations

- ComaCreator is derived from the Kramer et al. 67P dust/gas work but 'doesn't function as they intended, has more bugs certainly, and we have no collaboration with them' — proof-of-concept only.
- ComaCreator 'is in development stage maybe forever' and is intended only for realistic-looking pictures, not scientific fluxes.
- Large meshes need decimation (3M triangles 'will take forever'); increasing spatial resolution (e.g. switching to OpenVDB) and reducing EXR file size are open TODOs.
- Standalone ComaCreator is untested; the plugin path (interface.py, /app/ComaCreator) must be wired through Sispo input options.

## Evidence

- [readme] comet surface noise sources: [evidence span withheld — anchor status: external_doc]
- [readme] gas constants: [evidence span withheld — anchor status: external_doc]
- [readme] mesh decimation: [evidence span withheld — anchor status: external_doc]
- [readme] grid resolution: [evidence span withheld — anchor status: external_doc]
- [readme] scope caveat: [evidence span withheld — anchor status: external_doc]
- [abstract] SISPO comet modelling capability: "has supplementary models to produce realistic dust- and gas-environment optical models for comets and active asteroids"
- [readme] pipeline structure: [evidence span withheld — anchor status: external_doc]
