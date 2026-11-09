---
name: client-side-web-image-rendering
description: Use when a client must render hill-shaded relief from a 32-bit DEM height
  channel (stored as a 1-channel TIFF in an IIPImage multi-resolution pyramid) and
  the light-source incidence angle must be user-adjustable in real time.
when_to_use_negative:
- The input is already a rendered 8-bit compressed image — shading cannot be recomputed
  because per-pixel elevation and normal data were lost to channel mixing, scaling
  or compression.
- The goal is passive visualization of a static raster rather than interactive, angle-adjustable
  relief shading; plain IIP tile streaming suffices.
- The data is a catalog or point-source list (e.g. 2MASS, SDSS catalogs) rather than
  a gridded DEM elevation channel.
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: IIPImage (iipsrv)
  role: C++ FastCGI image server that performs dynamic hillshading of 3D surface topologies
    and streams 1/8/16/32-bit (including 32-bit float) tiled TIFF tiles to the web
    client; the SHD command's pipeline is the reference behavior being reproduced
  repo: https://github.com/cmarmo/iipsrv-astro
- name: JavaScript
  role: Client-side language implementing the Horn (1981) gradient estimation, normal
    packing and dot-product intensity modulation in a stock browser
- name: HTML5
  role: Standard the lightweight, platform-independent web client is built on, compatible
    with touch and mouse-based devices
- name: WebGL
  role: Interactive 3D-graphics API used to render the shaded relief from the streamed
    height/normal channels
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2014.12.006
    title: Web-based visualization of very large scientific astronomy imagery
  source_content_hash: sha256:7bfbfbd82c5272d960a6b22e5ebc82d0116675607722bef355c6dae2283c2e84
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/cmarmo/iipsrv-astro
  tools:
  - HTML5
  - IIPImage (iipsrv)
  - JavaScript
  - WebGL
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# client-side-web-image-rendering

## Summary

Reconstructs the IIPImage server-side hillshading (SHD) rendering of a DEM raster in the browser: convert elevation point data to per-pixel surface normals and heights, then modulate pixel intensity by the dot product of the normals with an adjustable illumination vector. This lets a web client reproduce the SHD output of the lightweight C++ IIPImage server without re-encoding the source data.

## When to use

Use when a client must render hill-shaded relief from a 32-bit DEM height channel (stored as a 1-channel TIFF in an IIPImage multi-resolution pyramid) and the light-source incidence angle must be user-adjustable in real time.

## When NOT to use

- The input is already a rendered 8-bit compressed image — shading cannot be recomputed because per-pixel elevation and normal data were lost to channel mixing, scaling or compression.
- The goal is passive visualization of a static raster rather than interactive, angle-adjustable relief shading; plain IIP tile streaming suffices.
- The data is a catalog or point-source list (e.g. 2MASS, SDSS catalogs) rather than a gridded DEM elevation channel.

## Inputs

- Full-precision 32-bit elevation channel of a DEM raster served by IIPImage
- 1-channel monochrome 32-bit TIFF height map with tileable multi-resolution pyramid structure
- 3-channel 'color' 32-bit TIFF of packed XYZ surface-normal vectors
- User-selected light-source incidence angle (numeric parameter)

## Outputs

- hill-shaded relief rendering displayed in the browser client (JavaScript/HTML5/WebGL)
- hillshade_render.png snapshot of the rendered display
- derived 32-bit TIFF tiles of per-pixel surface normals (3-channel) and heights (1-channel)

## How to apply

Load the full-precision 32-bit elevation channel from the server-side scientific image; these values are unaffected by channel mixing, image scaling or compression. Estimate per-pixel surface gradients over a 3x3 neighborhood using the Horn (1981) finite-difference scheme, and derive the surface normal components and height per pixel. Pack the XYZ normal vectors into a 3-channel 'color' 32-bit TIFF and the height data into a separate 1-channel monochrome TIFF, both tiled and structured into a multi-resolution pyramid for streaming with IIPImage. Render the hill-shaded relief by modulating pixel intensity with the dot product of surface normals and the illumination vector, exposing the light-source incidence angle as a numeric parameter (entered directly or via -/+ buttons), updating the display only on button release to keep client-server exchanges reasonable. Capture the result as a PNG via the snapshot mechanism.

## Related tools

- **IIPImage (iipsrv)** (C++ FastCGI image server that performs dynamic hillshading of 3D surface topologies and streams 1/8/16/32-bit (including 32-bit float) tiled TIFF tiles to the web client; the SHD command's pipeline is the reference behavior being reproduced) — https://github.com/cmarmo/iipsrv-astro
- **JavaScript** (Client-side language implementing the Horn (1981) gradient estimation, normal packing and dot-product intensity modulation in a stock browser)
- **HTML5** (Standard the lightweight, platform-independent web client is built on, compatible with touch and mouse-based devices)
- **WebGL** (Interactive 3D-graphics API used to render the shaded relief from the streamed height/normal channels)

## Examples

```
iipsrv.fcgi?FIF=dem.tif&SHD=90&WID=1024&CVT=jpeg
```

## Evaluation signals

- hillshade_render.png exists as a valid PNG produced via the snapshot mechanism
- Rendered shading varies monotonically with the selected light-source incidence angle
- The rendered output matches the reference hill-shaded rendering of the same channel (iipsrv SHD result)
- Packed normal channels form a valid 3-channel 32-bit TIFF and heights a 1-channel monochrome 32-bit TIFF, both tiled into the multi-resolution pyramid
- Per-pixel gradients are consistent with a 3x3-neighborhood Horn (1981) finite-difference estimate of the height map

## Limitations

- The image updates only when the incidence-angle buttons are released, to keep client-server exchanges reasonable; continuous dragging does not re-render.
- Browser engines display gamma-encoded 8-bit-per-channel images, so full bit depth must be preserved server-side / in the 32-bit TIFF pipeline; static 8-bit compressed storage limits interaction to passive visualization.
- Correctness depends on loading the full-precision 32-bit elevation values; any channel mixing, image scaling or compression corrupts the height input.
- The rendering is client-side reconstruction of server-side SHD logic — deviations from the Horn (1981) scheme or the IIP SHD parameterization will not match the reference rendering.

## Evidence

- [other] Elevation point data is converted to per-pixel vector normal and height data, which are stored in standard TIFF format: [evidence span withheld — anchor status: not_found]
- [readme] IIPImage README feature list on dynamic hillshading: [evidence span withheld — anchor status: external_doc]
- [readme] Bit-depth support in iipsrv: [evidence span withheld — anchor status: external_doc]
- [intro] Browser 8-bit display vs scientific bit depth: [evidence span withheld — anchor status: not_found]
- [intro] Client technology stack: [evidence span withheld — anchor status: not_found]
