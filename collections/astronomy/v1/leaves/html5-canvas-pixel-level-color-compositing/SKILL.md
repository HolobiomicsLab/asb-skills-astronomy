---
name: html5-canvas-pixel-level-color-compositing
description: 'Use when an interactive web client must composite multi-band planetary/astronomy
  tile images on demand, and per-tile processing time (target: 1–3 ms per 256×256
  tile) must be measured in-browser.'
when_to_use_negative:
- 'When the server should do the heavy lifting: the paper offloads server-side transcoding
  of FITS to browser-friendly formats because browser engines display gamma-encoded
  8-bit images while scientific data are linearly quantized 16-bit or floating point
  — do not use in-browser canvas compositing to substitute for that server-side encoding
  step.'
- When pixels are only passively visualized from static 8-bit compressed images (no
  on-demand mixing), the skill's per-tile getImageData/putImageData loop and benchmark
  are unnecessary.
- When per-tile timing must include network fetch time — the skill explicitly measures
  only the client-side composite step, excluding network fetch.
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_3318
tools:
- name: JavaScript
  role: Runs the per-tile getImageData/putImageData compositing loop and performance.now()
    benchmark in the browser.
- name: HTML5
  role: Provides the canvas 2D context used to read and write composited tile pixels.
- name: VisiOmatic
  role: Reference implementation whose Channel Mixing panel's Color Mixing (composite)
    mode defines the mixing-matrix parameters the canvas compositing must mirror.
  repo: https://github.com/astromatic/visiomatic
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ascom.2014.12.006
    title: Web-based visualization of very large scientific astronomy imagery
  source_content_hash: sha256:ad96283e3b2cd68cde41c532b8fbaf3cec2292cac0f17010bd032ea722b06c19
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/astromatic/visiomatic
  tools:
  - HTML5
  - JavaScript
  - VisiOmatic
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# html5-canvas-pixel-level-color-compositing

## Summary

Reconstruct the client-side, per-tile color compositing of multi-channel scientific imagery using the HTML5 canvas 2D context: read single-channel 256×256 tile pixels with getImageData, apply a user-adjustable channel mixing matrix, write composited RGBA back with putImageData, and benchmark the per-tile processing time in JavaScript.

## When to use

Use when an interactive web client must composite multi-band planetary/astronomy tile images on demand, and per-tile processing time (target: 1–3 ms per 256×256 tile) must be measured in-browser.

## When NOT to use

- When the server should do the heavy lifting: the paper offloads server-side transcoding of FITS to browser-friendly formats because browser engines display gamma-encoded 8-bit images while scientific data are linearly quantized 16-bit or floating point — do not use in-browser canvas compositing to substitute for that server-side encoding step.
- When pixels are only passively visualized from static 8-bit compressed images (no on-demand mixing), the skill's per-tile getImageData/putImageData loop and benchmark are unnecessary.
- When per-tile timing must include network fetch time — the skill explicitly measures only the client-side composite step, excluding network fetch.

## Inputs

- Set of single-channel 256×256 PNG tile images (e.g. HRSC Mars multi-band tiles)
- User-adjustable channel mixing matrix (per channel: R/G/B contribution, min/max channel levels, contrast/scaling factor)

## Outputs

- Composited RGBA canvas tile (putImageData output) visually matching the expected channel-color mix
- Per-tile and average compositing times written to compositing_benchmark (target 1–3 ms per 256×256 tile)

## How to apply

Load single-channel 256×256 PNG tiles (e.g. from the HRSC Mars multi-band example) as canvas-source bitmaps, then define the user-adjustable mixing matrix holding, per contributing channel, the target R/G/B contribution, the min/max channel levels, and the contrast scaling factor — mirroring the VisiOmatic Channel Mixing panel's Color Mixing (composite) mode. For each tile, read pixels via the canvas 2D context (getImageData), apply the linear interpolation of colors between the selected channels defined by the matrix, and write the composited RGBA output back with putImageData. Wrap each per-tile composite with performance.now() timers so fetch time is excluded, and repeat across many tiles to get a stable average per-tile compositing time; record per-tile and average values (e.g. to compositing_benchmark). Rationale: the paper's system offloads on-demand color compositing to the browser because modern JavaScript engines and HTML5 canvas make 1–3 ms per 256×256 tile achievable depending on browser and client hardware, keeping the lightweight C++ server free for server-side transcoding and streaming.

## Related tools

- **JavaScript** (Runs the per-tile getImageData/putImageData compositing loop and performance.now() benchmark in the browser.)
- **HTML5** (Provides the canvas 2D context used to read and write composited tile pixels.)
- **VisiOmatic** (Reference implementation whose Channel Mixing panel's Color Mixing (composite) mode defines the mixing-matrix parameters the canvas compositing must mirror.) — https://github.com/astromatic/visiomatic

## Examples

```
performance.now() around getImageData → matrix mix → putImageData per 256×256 PNG tile; repeat for stable average → compositing_benchmark
```

## Evaluation signals

- Reported per-tile compositing time must fall in the 1–3 ms range per 256×256 tile as reported for the browser-side implementation.
- Composited canvas output must visually match the expected channel-color mix defined by the mixing matrix.
- Timing must use performance.now() wrapping the composite step only, excluding network fetch time.
- Average compositing time must be computed over multiple tiles for a stable average.
- Per-tile and average compositing times must be recorded (e.g. to compositing_benchmark).

## Limitations

- Reported 1–3 ms per 256×256 tile is 'depending on browser and client hardware' — timings are hardware/browser dependent and must be averaged over multiple tiles.
- Browser engines display gamma-encoded 8-bit-per-channel images; linearly quantized 16-bit or floating-point scientific values still require server-side compression/encoding first, so canvas compositing alone cannot render full-precision data.
- The benchmark excludes network fetch time, so it does not reflect end-to-end tile load latency.

## Evidence

- [results] On-demand color compositing is performed in the browser via the HTML5 canvas: [evidence span withheld — anchor status: not_found]
- [readme] Canvas read/write API: [evidence span withheld — anchor status: not_found]
