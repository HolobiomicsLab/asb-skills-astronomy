---
name: opengl-renderer-performance-benchmarking
description: Use when you need wall-clock timing figures for SISPO image generation
  (per-image seconds) and closed-loop GNC mission-week runs on CPU-limited hardware,
  to decide whether the fast OpenGL renderer suffices instead of Cycles.
when_to_use_negative:
- The goal is photorealistic, physically based rendering quality (e.g., for imaging-instrument
  performance characterisation) rather than fast image generation — Cycles path-tracing
  is the appropriate renderer there.
- You need ComaCreator dust/gas coma environments rendered, since the ComaCreator
  plugin is invoked by Sispo with 'Only Cycles support' and cannot run on the OpenGL
  renderer path.
- The benchmark hardware differs substantially from the reference i7-7700HQ laptop
  and absolute (not relative) timing numbers are required.
tools:
- name: SISPO
  role: Provides the CLI and the OpenGL-based lightweight renderer whose image-generation
    and closed-loop GNC timings are benchmarked
  repo: https://github.com/SISPO-developers/sispo
- name: sispo_docker
  role: Docker-based deployment to install and run SISPO consistently for the benchmark
    runs
  repo: https://github.com/SISPO-developers/sispo_docker
- name: Cycles
  role: Photorealistic Blender path-tracing engine used as the slower alternative
    against which the OpenGL renderer's speed advantage is contextualised
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pone.0263882
    title: 'SISPO: Space Imaging Simulator for Proximity Operations'
  source_content_hash: sha256:3d39bedd492351a4f4375ede43a097ebb3e297b67d87a8da77c4db904246becf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SISPO-developers/sispo
  tools:
  - Cycles
  - SISPO
  - sispo_docker
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# opengl-renderer-performance-benchmarking

## Summary

Benchmarks SISPO's lightweight OpenGL-based renderer against the photorealistic Cycles path-tracer by timing per-image generation and closed-loop GNC wall-clock performance on consumer laptop hardware, quantifying the speed/photorealism trade-off for small-body mission simulation.

## When to use

Use when you need wall-clock timing figures for SISPO image generation (per-image seconds) and closed-loop GNC mission-week runs on CPU-limited hardware, to decide whether the fast OpenGL renderer suffices instead of Cycles.

## When NOT to use

- The goal is photorealistic, physically based rendering quality (e.g., for imaging-instrument performance characterisation) rather than fast image generation — Cycles path-tracing is the appropriate renderer there.
- You need ComaCreator dust/gas coma environments rendered, since the ComaCreator plugin is invoked by Sispo with 'Only Cycles support' and cannot run on the OpenGL renderer path.
- The benchmark hardware differs substantially from the reference i7-7700HQ laptop and absolute (not relative) timing numbers are required.

## Inputs

- 25143 Itokawa asteroid 3-D model (shape model used by the SISPO article)
- SISPO input configuration selecting the OpenGL-based renderer (not Cycles) and defining the fly-by camera trajectory
- Closed-loop GNC simulation setup spanning one simulated mission week

## Outputs

- render_benchmark.csv with per-image generation times and total mission-week wall-clock time
- Per-image timing distribution (~0.7–1.0 s per image on an Intel i7-7700HQ, 2.8 GHz)
- Mission-week wall-clock figure (~12 h on the reference laptop)

## How to apply

Install sispo as a Python package via pip so its CLI is available (or use the sispo_docker container, which is the recommended easiest way to run SISPO). Load the 25143 Itokawa asteroid 3-D model and configure the SISPO camera model to select the lightweight OpenGL-based renderer rather than Blender's Cycles path-tracing engine. Render the fly-by image sequence, recording the generation time of each individual image, then run the closed-loop GNC simulation over one simulated mission week and record total wall-clock time. Tabulate per-image generation times together with the mission-week wall-clock time in a render_benchmark.csv file. Interpret timings relative to the reference platform: an Intel i7-7700HQ (2.8 GHz) laptop, where one image took ~0.7–1.0 s and one simulated mission week took ~12 h; deviations outside these ranges on comparable hardware indicate configuration or environment problems.

## Related tools

- **SISPO** (Provides the CLI and the OpenGL-based lightweight renderer whose image-generation and closed-loop GNC timings are benchmarked) — https://github.com/SISPO-developers/sispo
- **sispo_docker** (Docker-based deployment to install and run SISPO consistently for the benchmark runs) — https://github.com/SISPO-developers/sispo_docker
- **Cycles** (Photorealistic Blender path-tracing engine used as the slower alternative against which the OpenGL renderer's speed advantage is contextualised)

## Examples

```
pip install sispo; sispo --with-render --with-sim
```

## Evaluation signals

- Per-image generation times on comparable hardware fall in the ~0.7–1.0 s range rather than the much longer Cycles path-tracing times.
- Closed-loop GNC wall-clock time for one simulated mission week is near ~12 h on an Intel i7-7700HQ (2.8 GHz) class laptop.
- render_benchmark.csv exists and contains both per-image timings and the mission-week total.
- The rendered image sequence uses the 25143 Itokawa model and the OpenGL renderer, confirmed in the SISPO camera/renderer configuration.
- Timings are recorded on hardware explicitly documented in the benchmark metadata (CPU model and clock speed).

## Limitations

- Timing figures are specific to the reference Intel i7-7700HQ (2.8 GHz) laptop and will differ on other hardware or under Docker VM constraints (e.g., limited VM cores/memory).
- The OpenGL renderer sacrifices the physically based rendering fidelity of Cycles, so results are not photorealistic.
- Using the tool requires basic programming skills, and the source repository has no changelog to track renderer performance changes across versions.

## Evidence

- [results] Per-image and mission-week timing figures on the reference laptop: [evidence span withheld — anchor status: not_found]
- [methods] SISPO CLI installation route for running the benchmark: "If sispo is installed as a python package via pip, sispo provides a CLI."
- [readme] Recommended deployment route via Docker: [evidence span withheld — anchor status: external_doc]
- [intro] Asteroid shape model used for the rendered image sequence: "The 25143 Itokawa asteroid model by the [14] was used."
