---
name: full-precision-scientific-image-handling
description: Use when scientific imagery stored as 32-bit floating point or 16-bit
  data must be visualized in a web browser, because browser engines only display gamma-encoded
  8-bit-per-channel images and quantization must be avoided server-side.
when_to_use_negative:
- The image is already a static 8-bit compressed graphic — interaction would be limited
  to passive visualization and no full-precision processing is possible
- A small image that fits in memory and can be fully decoded client-side from FITS
  via JavaScript (e.g. jsFITS) without server round-trips
- The target client requires lossless 32-bit pixel values in the browser — browser
  engines only display gamma-encoded 8-bit-per-channel images
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: iipsrv (IIPImage server)
  role: Lightweight C++ FastCGI server that transcodes and streams tiles from 8/16/32-bit
    (including 32-bit floating point) TIFF/JPEG2000 sources, with dynamic hillshading
    of 3D surface topologies
  repo: https://github.com/cmarmo/iipsrv-astro
- name: VisiOmatic
  role: Web client for visualizing streamed full-bit-depth astronomical image tiles
    served by iipsrv
  repo: https://github.com/astromatic/visiomatic
- name: jsFITS
  role: 'Alternative client-side path: JavaScript FITS reader that decodes FITS directly
    in the browser via canvas (used for small local files)'
  repo: https://github.com/slowe/jsFITS
- name: Mizar
  role: 3D browser application (WebGL) for visualizing geospatial/celestial data served
    via WMS and HiPS
  repo: https://github.com/TPZF/RTWeb3D
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2014.12.006
    title: Web-based visualization of very large scientific astronomy imagery
  source_content_hash: sha256:6ce965d6e3d82ad0d2cda47a715bee740e265f2e6966490030a7cdcbfa8cfd86
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/cmarmo/iipsrv-astro
  tools:
  - Mizar
  - VisiOmatic
  - WebGL
  - iipsrv (IIPImage server)
  - jsFITS
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# full-precision-scientific-image-handling

## Summary

Server-side handling of full bit depth (16-bit/32-bit floating point) scientific imagery so that browser clients never see quantization loss: data are transcoded to tileable multi-resolution formats (TIFF/JPEG2000) and streamed as compressed PNG/JPEG tiles from a lightweight C++ IIPImage server to HTML5/JavaScript clients.

## When to use

Use when scientific imagery stored as 32-bit floating point or 16-bit data must be visualized in a web browser, because browser engines only display gamma-encoded 8-bit-per-channel images and quantization must be avoided server-side.

## When NOT to use

- The image is already a static 8-bit compressed graphic — interaction would be limited to passive visualization and no full-precision processing is possible
- A small image that fits in memory and can be fully decoded client-side from FITS via JavaScript (e.g. jsFITS) without server round-trips
- The target client requires lossless 32-bit pixel values in the browser — browser engines only display gamma-encoded 8-bit-per-channel images

## Inputs

- Full precision 32-bit floating point astronomy imagery (e.g. FITS science data)
- 16-bit linearly quantized scientific images
- DEM height data as 32-bit TIFF tiles (elevation channel)
- Terabyte-scale image datasets (e.g. 2MASS Point Source Catalog imagery, SDSS Photometric Catalog imagery)

## Outputs

- Tileable multi-resolution TIFF/JPEG2000 pyramids with 32-bit float channels
- On-the-fly compressed PNG/JPEG tiles streamed to the web client
- Derived products such as 3-channel XYZ normal TIFFs plus 1-channel monochrome height TIFFs for hillshading
- Browser-rendered 8-bit visualization with real-time image setting adjustments

## How to apply

Load the full-precision (32-bit) pixel values from the server-side scientific image, which are unaffected by channel mixing, image scaling or compression. Transcode the original science data server-side to a browser-friendly tiled format (TIFF or JPEG2000) structured into a multi-resolution pyramid, rather than decoding FITS in the browser via JavaScript. Stream compressed tiles on-the-fly from the lightweight C++ IIPImage server (iipsrv) using the IIP/Zoomify/DeepZoom/IIIF protocols, applying dynamic image settings (e.g. hillshading via per-pixel vector normals stored in a 3-channel TIFF plus height in a monochrome TIFF, or channel mixing) at request time. Keep the client as a lightweight W3C-standards JavaScript application built on HTML5/CSS3/WebGL that only receives 8-bit compressed tiles. Rationale: static 8-bit images limit interaction to passive visualization, and the increasing gap between storage capacity and data access bandwidth makes it efficient to offload image processing to the server.

## Related tools

- **iipsrv (IIPImage server)** (Lightweight C++ FastCGI server that transcodes and streams tiles from 8/16/32-bit (including 32-bit floating point) TIFF/JPEG2000 sources, with dynamic hillshading of 3D surface topologies) — https://github.com/cmarmo/iipsrv-astro
- **VisiOmatic** (Web client for visualizing streamed full-bit-depth astronomical image tiles served by iipsrv) — https://github.com/astromatic/visiomatic
- **jsFITS** (Alternative client-side path: JavaScript FITS reader that decodes FITS directly in the browser via canvas (used for small local files)) — https://github.com/slowe/jsFITS
- **Mizar** (3D browser application (WebGL) for visualizing geospatial/celestial data served via WMS and HiPS) — https://github.com/TPZF/RTWeb3D

## Examples

```
iipsrv.fcgi?FIF=image_32bit.tif&SHD=45&WID=1024&CVT=jpeg
```

## Evaluation signals

- Requested pixel values match the full-precision (32-bit) source channel — unaffected by channel mixing, image scaling or compression
- Output tiles form a consistent multi-resolution pyramid whose coarser levels are downsampled versions of finer levels
- A single server comfortably serves more than a hundred simultaneous users accessing full precision 32-bit astronomy data without degraded interactivity
- For hillshading: rendered shading varies monotonically with the light-source incidence angle parameter, matching a reference hill-shaded rendering of the same channel
- Client remains lightweight: HTML5/CSS3/JavaScript running in stock browsers on both touch and mouse-based devices

## Limitations

- Browser engines display gamma-encoded 8-bit-per-channel images, so server-side compression and encoding of the original data to a browser-friendly format is always necessary
- Storing data as static 8-bit compressed images limits interaction to passive visualization; dynamic server-side processing is required instead
- JPEG2000 decoding via OpenJPEG is experimental and much slower than the proprietary Kakadu SDK, and is disabled by default
- Image settings update only when adjustment buttons are released, to keep client-server exchanges reasonable
- The increasing gap between storage capacity and data access bandwidth constrains how much processing can be moved to the client

## Evidence

- [intro] Static 8-bit limitation: [evidence span withheld — anchor status: not_found]
- [readme] Server capability: [evidence span withheld — anchor status: not_found]
- [intro] Transcoding rationale: [evidence span withheld — anchor status: not_found]
