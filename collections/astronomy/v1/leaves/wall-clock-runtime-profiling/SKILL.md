---
name: wall-clock-runtime-profiling
description: Use when you need concrete wall-clock performance numbers (per-image
  render time and multi-day mission simulation runtime) for SISPO's lightweight OpenGL
  renderer on laptop-class hardware such as an Intel i7-7700HQ.
when_to_use_negative:
- When the goal is image quality/fidelity assessment rather than timing — use SISPO's
  image-comparison script to compare Cycles photorealistic output instead of profiling
  wall-clock
- When benchmarking the Cycles path-tracing engine itself, whose runtimes are expected
  to be far longer than the ~0.7–1.0 s per-image OpenGL figures
- When wall-clock latency is irrelevant and only output image content matters
tools:
- name: SISPO
  role: Provides the CLI and renderer configuration used to time per-image generation
    and closed-loop GNC simulation
  repo: https://github.com/SISPO-developers/sispo
- name: sispo_docker
  role: Alternative Docker-based installation path to deploy SISPO quickly for reproducible
    benchmark runs
  repo: https://github.com/SISPO-developers/sispo_docker
- name: Blender/Cycles
  role: Photorealistic path-tracing engine that the lightweight OpenGL renderer is
    benchmarked against (explicitly not used for the timed runs)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1371/journal.pone.0263882
    title: 'SISPO: Space Imaging Simulator for Proximity Operations'
  source_content_hash: sha256:5f60eaf5d5abf189cc3635381e5925167ed5321ee1f2a76677701a4d86e7eada
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SISPO-developers/sispo
  tools:
  - Blender/Cycles
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

# wall-clock-runtime-profiling

## Summary

Benchmarks SISPO's lightweight OpenGL-based renderer by timing per-image generation and closed-loop GNC simulation wall-clock runtime on CPU-class hardware, quantifying the speed trade-off against the photorealistic Cycles path-tracer.

## When to use

Use when you need concrete wall-clock performance numbers (per-image render time and multi-day mission simulation runtime) for SISPO's lightweight OpenGL renderer on laptop-class hardware such as an Intel i7-7700HQ.

## When NOT to use

- When the goal is image quality/fidelity assessment rather than timing — use SISPO's image-comparison script to compare Cycles photorealistic output instead of profiling wall-clock
- When benchmarking the Cycles path-tracing engine itself, whose runtimes are expected to be far longer than the ~0.7–1.0 s per-image OpenGL figures
- When wall-clock latency is irrelevant and only output image content matters

## Inputs

- 25143 Itokawa asteroid 3-D model (shape model used by SISPO)
- SISPO input configuration with fly-by trajectory and camera intrinsic parameters, camera set to the OpenGL-based renderer
- Closed-loop GNC simulation scenario spanning one simulated mission week
- Host hardware comparable to an Intel i7-7700HQ (2.8 GHz) laptop

## Outputs

- Per-image generation times for the rendered fly-by image sequence
- Total wall-clock time for one simulated mission week of closed-loop GNC
- render_benchmark.csv tabulating the measured timings

## How to apply

Install sispo as a Python package via pip so its CLI is available (or use the sispo_docker image). Load the 25143 Itokawa asteroid 3-D model and configure the SISPO camera model to use the lightweight OpenGL-based renderer instead of the photorealistic Cycles path-tracing engine — this substitution is the whole point of the benchmark, since Cycles is the slower physically based path. Render the image sequence for the fly-by scenario, recording the generation time of each individual image. Then run the closed-loop GNC simulation over one simulated mission week with the same lightweight renderer and record the total wall-clock time. Tabulate the per-image generation times and the mission-week wall-clock time into render_benchmark.csv. Judge success by comparing against the reported reference: ~0.7–1.0 s per image and ~12 h per simulated mission week on an Intel i7-7700HQ (2.8 GHz).

## Related tools

- **SISPO** (Provides the CLI and renderer configuration used to time per-image generation and closed-loop GNC simulation) — https://github.com/SISPO-developers/sispo
- **sispo_docker** (Alternative Docker-based installation path to deploy SISPO quickly for reproducible benchmark runs) — https://github.com/SISPO-developers/sispo_docker
- **Blender/Cycles** (Photorealistic path-tracing engine that the lightweight OpenGL renderer is benchmarked against (explicitly not used for the timed runs))

## Examples

```
pip install sispo; sispo --with-render --with-sim  # OpenGL renderer; record per-image and mission-week timings into render_benchmark.csv
```

## Evaluation signals

- Per-image generation time falls near the reported 0.7–1.0 s range on i7-7700HQ-class hardware
- One simulated mission week of closed-loop GNC completes in roughly 12 h wall-clock on comparable hardware
- render_benchmark.csv contains both per-image times and the mission-week total, confirming the loop recorded each image individually
- Camera configuration confirms the lightweight OpenGL renderer was active (not Cycles) during the timed run
- Timings are reported alongside a hardware specification (CPU model and clock) so results are comparable across machines

## Limitations

- Timings are hardware-dependent; the reference figures are specific to an Intel i7-7700HQ (2.8 GHz) laptop and will differ on other machines
- SISPO is open access but currently requires basic programming skills to use, so benchmark setup is not turnkey
- No changelog is available for the source repository, so version drift between benchmark runs may not be traceable

## Evidence

- [full_text] reported per-image and mission-week timings: [evidence span withheld — anchor status: not_found]
- [methods] CLI availability via pip install: "If sispo is installed as a python package via pip, sispo provides a CLI."
- [intro] Blender/Cycles is the default rendering path being contrasted: "SISPO uses Cycles rendering with the Blender software package, which allows for programming procedures in Python"
- [discussion] Cycles path-tracing as the slower photorealistic baseline: [evidence span withheld — anchor status: not_found]
