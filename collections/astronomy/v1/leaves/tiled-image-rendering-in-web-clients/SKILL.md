---
name: tiled-image-rendering-in-web-clients
description: Use when a browser client must render or interactively re-color very
  large scientific images (terabyte-scale full-bit-depth astronomy or planetary data)
  without a server round-trip per view change, e.g. for channel-mixing or zoom/pan
  tile display.
when_to_use_negative:
- When pixel values must remain full precision for quantitative analysis — browsers
  display gamma-encoded 8-bit-per-channel images, so server-side compression/encoding
  to a browser-friendly format is required and client rendering is for visualization
  only
- When data are stored as static 8-bit compressed images with no server-side dynamic
  processing/tiling — interaction is then limited to passive visualization and on-demand
  compositing cannot work
- When cross-domain fetches are involved — browser security restrictions require tiles
  hosted on the same domain as the client (per the jsFITS README)
edam_operation: http://edamontology.org/operation_0337
tools:
- name: VisiOmatic
  role: Reference client whose Channel Mixing panel (Color Mixing / composite mode)
    defines the per-tile canvas compositing matrix; source at github.com/astromatic/visiomatic
  repo: https://github.com/astromatic/visiomatic
- name: iipsrv-astro (IIPImage)
  role: FastCGI streaming image server generating the 256×256 compressed PNG/JPEG2000
    tiles fetched by the client, with 16/32-bit and multispectral support
  repo: https://github.com/cmarmo/iipsrv-astro
- name: jsFITS
  role: 'Alternative JavaScript route: read FITS files client-side and display via
    <canvas>, useful when running locally since FITS files are uncompressed and large'
  repo: https://github.com/slowe/jsFITS
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.ascom.2014.12.006
    title: Web-based visualization of very large scientific astronomy imagery
  source_content_hash: sha256:492b52de7e7b4bfed30e0bf2355d267a13aa1e4105501f2ee8b4d0d6f409a94f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/astromatic/visiomatic
  tools:
  - VisiOmatic
  - iipsrv-astro (IIPImage)
  - jsFITS
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# tiled-image-rendering-in-web-clients

## Summary

Client-side tiled rendering of large scientific imagery: 256×256 pixel PNG tiles (e.g. from HRSC Mars multi-band data or JPEG2000-encoded astronomy sources) are composited in the browser with the HTML5 canvas 2D context, taking ~1–3 ms per tile for on-demand color mixing.

## When to use

Use when a browser client must render or interactively re-color very large scientific images (terabyte-scale full-bit-depth astronomy or planetary data) without a server round-trip per view change, e.g. for channel-mixing or zoom/pan tile display.

## When NOT to use

- When pixel values must remain full precision for quantitative analysis — browsers display gamma-encoded 8-bit-per-channel images, so server-side compression/encoding to a browser-friendly format is required and client rendering is for visualization only
- When data are stored as static 8-bit compressed images with no server-side dynamic processing/tiling — interaction is then limited to passive visualization and on-demand compositing cannot work
- When cross-domain fetches are involved — browser security restrictions require tiles hosted on the same domain as the client (per the jsFITS README)

## Inputs

- Single-channel 256×256 PNG tile images (browser-friendly format, transcoded from JPEG2000-encoded or FITS science data)
- User-adjustable mixing matrix per channel (RGB contribution, min/max channel levels, contrast/scaling factor)
- Full-bit-depth source imagery (16-bit or 32-bit floating point, up to terabyte scale)

## Outputs

- Composited canvas RGBA output per 256×256 tile
- Per-tile compositing times and average (expected ~1–3 ms per 256×256 tile, written to a compositing_benchmark output)

## How to apply

Serve the large image as compressed PNG tiles generated on-the-fly server-side from JPEG2000-encoded data, and fetch 256×256 tiles directly from the browser. Load each tile as a canvas-source bitmap, then use the canvas 2D context's getImageData to read pixels and apply the user-adjustable channel mixing matrix (per-channel RGB contribution, min/max levels, contrast scaling factor, mirroring the VisiOmatic Channel Mixing panel) before writing composited RGBA pixels back with putImageData. Wrap each per-tile composite with performance.now timers, excluding network fetch time, and average over multiple tiles to obtain a stable per-tile compositing time. Server-side transcoding remains necessary because browser engines display gamma-encoded 8-bit-per-channel images while scientific data are linearly quantized 16-bit or floating point.

## Related tools

- **VisiOmatic** (Reference client whose Channel Mixing panel (Color Mixing / composite mode) defines the per-tile canvas compositing matrix; source at github.com/astromatic/visiomatic) — https://github.com/astromatic/visiomatic
- **iipsrv-astro (IIPImage)** (FastCGI streaming image server generating the 256×256 compressed PNG/JPEG2000 tiles fetched by the client, with 16/32-bit and multispectral support) — https://github.com/cmarmo/iipsrv-astro
- **jsFITS** (Alternative JavaScript route: read FITS files client-side and display via <canvas>, useful when running locally since FITS files are uncompressed and large) — https://github.com/slowe/jsFITS

## Examples

```
const ctx=canvas.getContext('2d'),t0=performance.now();const d=ctx.getImageData(0,0,256,256);ctx.putImageData(composite(d,mixingMatrix),0,0);console.log(performance.now()-t0);
```

## Evaluation signals

- Reported per-tile compositing time falls in the 1–3 ms range per 256×256 tile, as reported for the browser-side compositing implementation
- Composited canvas output visually matches the expected channel-color mix defined by the mixing matrix
- Timing measurements exclude network fetch time (timers wrap only the getImageData/putImageData composite step)
- Average over multiple tiles is stable, not a single-tile measurement
- Composited output matches across different browsers/hardware within the expected variance (compositing is measured per browser and client hardware)

## Limitations

- Per-tile processing time depends on browser and client hardware, so absolute benchmarks are not portable
- Browser engines display gamma-encoded 8-bit-per-channel images, whereas scientific images require linearly quantized 16-bit or floating point values — server-side compression and encoding remain necessary
- Static 8-bit compressed tiles limit interaction to passive visualization unless dynamic server-side processing and streaming are added
- Web browser cross-domain security issues restrict FITS/tile files to the same domain as the client (jsFITS README)

## Evidence

- [other] Per-tile compositing timing: [evidence span withheld — anchor status: not_found]
- [intro] Server-side tile generation: [evidence span withheld — anchor status: not_found]
- [readme] Canvas display in jsFITS: [evidence span withheld — anchor status: not_found]
- [readme] IIPImage tile server capability: [evidence span withheld — anchor status: external_doc]
