---
name: closed-loop-gnc-simulation-timing
description: Use when estimating whether SISPO's non-photorealistic OpenGL rendering
  path can support closed-loop GNC or visual-navigation studies within a practical
  wall-clock budget, as opposed to the Cycles path-tracer.
when_to_use_negative:
- The goal is photorealistic physically based rendering quality (Cycles path-tracing),
  where wall-clock speed is not the priority
- The scenario requires ComaCreator dust/gas environment plugins, which are invoked
  through Cycles rendering only
- The target is renderer output fidelity rather than performance benchmarking
tools:
- name: SISPO
  role: Provides the CLI, camera model, and the lightweight OpenGL rendering path
    used for the closed-loop GNC timing benchmark
  repo: https://github.com/SISPO-developers/sispo
- name: Blender
  role: 3-D visualisation system underlying SISPO; its Cycles engine is the photorealistic
    alternative being bypassed in favour of OpenGL
- name: sispo_docker
  role: Alternative Docker-based installation to simplify environment setup before
    running the benchmark
  repo: https://github.com/SISPO-developers/sispo_docker
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pone.0263882
    title: 'SISPO: Space Imaging Simulator for Proximity Operations'
  source_content_hash: sha256:e5b3e10acc230613099156b65bdb657e45a54534f65d31670315c73ab289f8e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SISPO-developers/sispo
  tools:
  - Blender
  - SISPO
  - sispo_docker
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# closed-loop-gnc-simulation-timing

## Summary

Benchmarks SISPO's lightweight OpenGL-based image renderer inside a closed-loop GNC analysis, measuring per-image generation time and total wall-clock time for one simulated mission week on laptop-class hardware.

## When to use

Use when estimating whether SISPO's non-photorealistic OpenGL rendering path can support closed-loop GNC or visual-navigation studies within a practical wall-clock budget, as opposed to the Cycles path-tracer.

## When NOT to use

- The goal is photorealistic physically based rendering quality (Cycles path-tracing), where wall-clock speed is not the priority
- The scenario requires ComaCreator dust/gas environment plugins, which are invoked through Cycles rendering only
- The target is renderer output fidelity rather than performance benchmarking

## Inputs

- Asteroid 3-D shape model (e.g., 25143 Itokawa model)
- SISPO input file configuring camera intrinsics, trajectory, and light parameters
- Renderer selection setting pointing to the OpenGL lightweight renderer rather than Cycles
- Closed-loop GNC simulation configuration spanning one simulated mission week

## Outputs

- render_benchmark.csv containing per-image generation times and the total mission-week wall-clock time
- Rendered image sequence for the fly-by scenario

## How to apply

Install sispo as a Python package via pip so that its CLI is available (or use the sispo_docker image). Load the 25143 Itokawa asteroid 3-D model and configure the SISPO camera model to select the lightweight OpenGL-based renderer instead of the Blender/Cycles path-tracer. Render the fly-by image sequence, recording the generation time of each individual image, then run the closed-loop GNC simulation over one simulated mission week with the same renderer, recording total wall-clock time. The published reference point is approximately 0.7–1.0 seconds per image and roughly 12 hours per simulated mission week on an Intel i7-7700HQ (2.8 GHz) laptop, so timings within that order of magnitude on comparable hardware indicate correct application. Tabulate per-image times and the mission-week wall-clock total and write them to render_benchmark.csv.

## Related tools

- **SISPO** (Provides the CLI, camera model, and the lightweight OpenGL rendering path used for the closed-loop GNC timing benchmark) — https://github.com/SISPO-developers/sispo
- **Blender** (3-D visualisation system underlying SISPO; its Cycles engine is the photorealistic alternative being bypassed in favour of OpenGL)
- **sispo_docker** (Alternative Docker-based installation to simplify environment setup before running the benchmark) — https://github.com/SISPO-developers/sispo_docker

## Examples

```
pip install sispo; sispo --with-render --with-sim
```

## Evaluation signals

- Per-image generation time falls around 0.7–1.0 s on hardware comparable to an Intel i7-7700HQ (2.8 GHz)
- One simulated mission week of closed-loop GNC completes in roughly 12 hours wall-clock on comparable hardware
- render_benchmark.csv is populated with per-image times and the mission-week total
- Renders originate from the OpenGL path, not Cycles, confirming the lightweight renderer was actually selected
- The rendered sequence matches the Itokawa fly-by trajectory/camera configuration in the SISPO input file

## Limitations

- Timings are anchored to laptop-class hardware (Intel i7-7700HQ, 2.8 GHz); other machines will scale differently
- SISPO is open access but requires basic programming skills to use, so benchmark setup is not turnkey
- Only the Cycles path supports ComaCreator plugin rendering; the OpenGL timing benchmark excludes dust/gas environments

## Evidence

- [results] per-image and mission-week timings: [evidence span withheld — anchor status: not_found]
- [methods] CLI installation: "If sispo is installed as a python package via pip, sispo provides a CLI."
- [intro] renderer alternatives and programmability: "SISPO uses Cycles rendering with the Blender software package, which allows for programming procedures in Python"
- [discussion] usability limitation: "The tool is open access and currently requires basic programming skills to be used"
