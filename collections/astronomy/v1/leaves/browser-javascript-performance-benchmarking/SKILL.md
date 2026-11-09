---
name: browser-javascript-performance-benchmarking
description: Use when you need to quantify the client-side cost of a browser-based
  pixel operation (tile compositing, channel mixing, filtering) on scientific imagery
  and must exclude network fetch time from the measurement.
when_to_use_negative:
- The operation runs server-side (e.g. JPEG2000-to-PNG tile generation in the C++
  server), since this skill measures browser/client-side JavaScript execution only
- Network latency or tile transfer throughput is the quantity of interest — fetch
  time is deliberately excluded from the per-tile timing
tools:
- name: JavaScript
  role: Host language for the per-tile compositing loop and the performance.now()
    timing wrappers
- name: HTML5
  role: Canvas 2D context (getImageData/putImageData) used to read and write composited
    RGBA pixels per tile
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ascom.2014.12.006
    title: Web-based visualization of very large scientific astronomy imagery
  source_content_hash: sha256:7d68c8bdadcd4587a1445cf5eb5b5b2682f3028237055ad4e3af7e8dbe4c11cf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - HTML5
  - JavaScript
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# browser-javascript-performance-benchmarking

## Summary

Benchmarks client-side JavaScript image-processing operations (e.g. per-tile HTML5 canvas color compositing) by wrapping individual tile operations with performance timers and averaging over many tiles, yielding per-operation timings like the ~1–3 ms per 256×256 tile reported for browser-side color compositing.

## When to use

Use when you need to quantify the client-side cost of a browser-based pixel operation (tile compositing, channel mixing, filtering) on scientific imagery and must exclude network fetch time from the measurement.

## When NOT to use

- The operation runs server-side (e.g. JPEG2000-to-PNG tile generation in the C++ server), since this skill measures browser/client-side JavaScript execution only
- Network latency or tile transfer throughput is the quantity of interest — fetch time is deliberately excluded from the per-tile timing

## Inputs

- Set of single-channel 256x256 PNG tile images (e.g. HRSC Mars multi-band tiles)
- User-adjustable channel mixing matrix (per-channel R/G/B contribution, min/max levels, contrast scaling factor)

## Outputs

- Per-tile compositing times (ms per 256x256 tile)
- Average per-tile compositing time written to a benchmark file (compositing_benchmark)

## How to apply

Load the single-channel PNG tiles (e.g. 256×256 tiles from a multi-band Mars HRSC example) into the browser as canvas-source bitmaps, then define the operation under test — here a user-adjustable channel-mixing matrix holding, per contributing channel, the target Red/Green/Blue contribution, minimum/maximum channel levels, and the contrast (scaling) factor, as in the VisiOmatic Channel Mixing panel's Color Mixing (composite) mode. For each tile, read pixels with the canvas 2D context (getImageData), apply the linear interpolation of colors between the selected channels defined by the mixing matrix, and write the composited RGBA pixels back (putImageData). Wrap each per-tile operation with JavaScript performance timers (performance.now) so every tile is timed individually and network fetch time is excluded. Repeat over many tiles to obtain a stable average per-tile time, then write per-tile and average values to a benchmark output file (e.g. compositing_benchmark).

## Related tools

- **JavaScript** (Host language for the per-tile compositing loop and the performance.now() timing wrappers)
- **HTML5** (Canvas 2D context (getImageData/putImageData) used to read and write composited RGBA pixels per tile)

## Examples

```
const t0 = performance.now(); ctx.putImageData(compositeTile(imgData, mixMatrix), 0, 0); logTileTime(performance.now() - t0);
```

## Evaluation signals

- Reported per-tile compositing time falls in the 1–3 ms range per 256×256 tile, as reported for the browser-side compositing implementation
- Composited canvas output visually matches the expected channel-color mix defined by the mixing matrix
- Timing excludes network fetch time (only the getImageData → mixing → putImageData span is measured per tile)
- Average is stable across repeated measurements over multiple tiles

## Limitations

- Per-tile timing depends on browser and client hardware, so absolute numbers are not portable across clients
- Benchmark applies to small (256×256) tiles; extrapolation to other tile sizes or whole-image compositing is not validated here
- Browser engines display gamma-encoded 8-bit-per-channel images, so full 16-bit/float precision must still be handled server-side before the client-side measurement is meaningful

## Evidence

- [other] per-tile compositing benchmark: [evidence span withheld — anchor status: not_found]
- [other] timing method: [evidence span withheld — anchor status: not_found]
- [intro] browser engine capability: [evidence span withheld — anchor status: not_found]
- [readme] canvas rendering in browser: [evidence span withheld — anchor status: not_found]
