---
name: tiled-image-pyramid-serving
description: Use when a web client must interactively browse full bit depth (16-bit
  or 32-bit float) images too large to transmit whole, and pixels must be re-processed
  server-side (scaling, colormap) rather than shipped as static 8-bit files.
when_to_use_negative:
- The image is already a small static 8-bit compressed file that can be shipped whole
  — interaction would be limited to passive visualization anyway, so no server-side
  tiling is needed
- The client is a desktop astronomy viewer (SAOimage, ds9, Gaia, Aladin) that can
  read FITS directly rather than a stock web browser
- Only 3D/scene rendering over geospatial or planetary data is required — a WebGL
  client such as MIZAR (RTWeb3D) addresses that use case instead of 2D tile streaming
edam_operation: http://edamontology.org/operation_0242
tools:
- name: visiomatic
  role: Open-source client-server system whose server component (visiomatic.server.tiled)
    serves tiles from the tiled pyramid TIFF via FastCGI to HTML5/JavaScript clients
  repo: github.com/astromatic/visiomatic
- name: iipsrv (IIPImage, cmarmo/iipsrv-astro)
  role: FastCGI image streaming server alternative supporting 8, 16 and 32 bits per
    channel including 32-bit floating point, gigapixel images and configurable tile
    cache
  repo: https://github.com/cmarmo/iipsrv-astro
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ascom.2014.12.006
    title: Web-based visualization of very large scientific astronomy imagery
  source_content_hash: sha256:6fb051189804adbdaa981d4b667647193904da227e3f5d7d996e6b233acf7e42
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - iipsrv (IIPImage, cmarmo/iipsrv-astro)
  - visiomatic
  edam_operation: http://edamontology.org/operation_0242
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# tiled-image-pyramid-serving

## Summary

Serve very large scientific astronomy imagery (e.g. 32-bit floating-point FITS-derived data at terabyte scale) as 256×256 image tiles from a tiled pyramid TIFF via a lightweight C++/FastCGI tile server, so stock HTML5 web browsers can pan and zoom full-precision images in real time.

## When to use

Use when a web client must interactively browse full bit depth (16-bit or 32-bit float) images too large to transmit whole, and pixels must be re-processed server-side (scaling, colormap) rather than shipped as static 8-bit files.

## When NOT to use

- The image is already a small static 8-bit compressed file that can be shipped whole — interaction would be limited to passive visualization anyway, so no server-side tiling is needed
- The client is a desktop astronomy viewer (SAOimage, ds9, Gaia, Aladin) that can read FITS directly rather than a stock web browser
- Only 3D/scene rendering over geospatial or planetary data is required — a WebGL client such as MIZAR (RTWeb3D) addresses that use case instead of 2D tile streaming

## Inputs

- Large scientific astronomy image (FITS or 16/32-bit per channel raster) at terabyte scale
- Tiled pyramid TIFF produced by multi-resolution tiling of the source image
- Tile HTTP requests from a browser client (zoom level, tile coordinates, image settings such as colormap/scaling)

## Outputs

- 256×256 image tiles streamed over HTTP in a browser-friendly format (e.g. PNG/JPEG)
- Tiled pyramid TIFF on the server filesystem as the multi-resolution serving representation
- Real-time adjusted views (server-side scaling/colormap applied per tile request)

## How to apply

First transcode the original FITS science data into a tiled pyramid TIFF, since browsers only display gamma-encoded 8-bit channels and server-side compression/encoding to a browser-friendly format remains necessary. Deploy the visiomatic tile server (the visiomatic.server.tiled module) behind a FastCGI-enabled web server such as Apache, pointing it at the tiled pyramid TIFF; the server generates 256×256 tiles on-the-fly and streams them as browser-friendly formats to the W3C-standards JavaScript client. Size the deployment using the benchmark result that a single 12-core server sustains ~700 concurrent 256×256 tile requests (about 100 users frantically browsing large uncompressed single-channel floating-point images), with single-threaded FastCGI per-tile request times of 5-10 ms as the baseline. Validate by load-testing (e.g. with a modified Apache Bench client) and confirming that requests-per-second stays approximately flat as concurrency rises while per-request latency grows linearly.

## Related tools

- **visiomatic** (Open-source client-server system whose server component (visiomatic.server.tiled) serves tiles from the tiled pyramid TIFF via FastCGI to HTML5/JavaScript clients) — github.com/astromatic/visiomatic
- **iipsrv (IIPImage, cmarmo/iipsrv-astro)** (FastCGI image streaming server alternative supporting 8, 16 and 32 bits per channel including 32-bit floating point, gigapixel images and configurable tile cache) — https://github.com/cmarmo/iipsrv-astro

## Examples

```
pip install visiomatic
```

## Evaluation signals

- Serving rate (requests/s) curve stays approximately flat as concurrent tile requests increase, reproducing the paper's Figure 8 behavior
- Per-request latency grows linearly with concurrency rather than collapsing, at least up to ~700 concurrent 256×256 tile requests
- System sustains ~700 concurrent tile requests (~100 users) on a single 12-core server with full precision 32-bit astronomy data
- Tiles are 256×256 and generated on-the-fly from the tiled pyramid TIFF, with single-threaded FastCGI per-tile time in the 5-10 ms range
- Image settings (scaling/colormap) adjust in real time in the client, confirming server-side dynamic processing rather than passive static tiles

## Limitations

- Browsers render gamma-encoded 8-bit-per-channel imagery, so server-side compression and encoding of the original data to a browser-friendly format is mandatory; native full-precision rendering in-browser is not achievable
- The ~700-concurrent-request capacity figure was measured for large, uncompressed, single-channel floating-point images on one 12-core server; other data types or hardware will scale differently
- Storing data as static 8-bit compressed images limits interaction to passive visualization — the pyramid/tiling server is required to lift that restriction, adding server-side compute per request

## Evidence

- [readme] Fast lightweight embeddable FastCGI server module: [evidence span withheld — anchor status: external_doc]
- [intro] having the data stored as static 8-bit compressed images means that interaction with the pixels is essentially limited to passive visualization: [evidence span withheld — anchor status: not_found]
