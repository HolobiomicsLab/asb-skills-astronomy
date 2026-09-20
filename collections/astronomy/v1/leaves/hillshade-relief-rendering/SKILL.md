---
name: hillshade-relief-rendering
description: Use when the input is a digital elevation model (DEM) height channel
  stored as full 32-bit TIFF tiles in an IIPImage multi-resolution pyramid and the
  goal is a hill-shaded relief rendering with a user-adjustable light-source incidence
  angle.
when_to_use_negative:
- The input is a plain RGB or 8-bit browser-format image with no 32-bit elevation/height
  channel — server-side transcoding of full bit depth data is required first.
- The goal is photometric visualization of astronomy imagery (e.g. FITS pixel rendering,
  color mixing, contrast/zoom) rather than 3D surface-topology relief shading.
- Only a static 8-bit compressed rendition of the terrain is available, since interaction
  would be limited to passive visualization.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: IIPImage (iipsrv)
  role: C++ FastCGI image server providing dynamic hillshading of 3D surface topologies
    and 32-bit floating point TIFF tile streaming via the SHD command
  repo: https://github.com/cmarmo/iipsrv-astro
- name: JavaScript
  role: Web-client implementation of the hillshade workflow (normal/height packing,
    SHD parameter updates on button release)
- name: HTML5
  role: Standards-based client framework for the browser-rendered hillshade interface
- name: WebGL
  role: Interactive 3D-graphics API supporting client-side rendering of the shaded
    relief
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2014.12.006
    title: Web-based visualization of very large scientific astronomy imagery
  source_content_hash: sha256:3b2f14d030474d6b218315a4b391e311e5fb7affceef3f16f0960bcdee1273b4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/cmarmo/iipsrv-astro
  tools:
  - HTML5
  - IIPImage (iipsrv)
  - JavaScript
  - WebGL
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# hillshade-relief-rendering

## Summary

Server-side reconstruction of hill-shaded relief from DEM height data: height pixels are converted to per-pixel surface normals and shaded by the dot product with an adjustable illumination vector, following the IIPImage SHD (hillshading) pipeline.

## When to use

Use when the input is a digital elevation model (DEM) height channel stored as full 32-bit TIFF tiles in an IIPImage multi-resolution pyramid and the goal is a hill-shaded relief rendering with a user-adjustable light-source incidence angle.

## When NOT to use

- The input is a plain RGB or 8-bit browser-format image with no 32-bit elevation/height channel — server-side transcoding of full bit depth data is required first.
- The goal is photometric visualization of astronomy imagery (e.g. FITS pixel rendering, color mixing, contrast/zoom) rather than 3D surface-topology relief shading.
- Only a static 8-bit compressed rendition of the terrain is available, since interaction would be limited to passive visualization.

## Inputs

- DEM elevation height map as full-precision 32-bit pixel values from a server-side scientific image
- 3-channel XYZ normal TIFF and 1-channel monochrome height TIFF (tiled, multi-resolution pyramid)
- User-selected light-source incidence angle (numeric parameter)

## Outputs

- Hill-shaded relief image (hillshade_render.png snapshot)
- Per-pixel vector normal (3-channel TIFF) and height (1-channel TIFF) products

## How to apply

Load the full-precision (32-bit) elevation channel from the server-side scientific image, which is unaffected by channel mixing, image scaling, or compression. Estimate per-pixel surface gradients over a 3x3 neighborhood using the Horn (1981) finite-difference scheme, then derive surface normal components and height per pixel. Pack the XYZ normals into a 3-channel 'color' TIFF and the height data into a separate 1-channel monochrome TIFF, both tileable and structured into a multi-resolution pyramid for streaming with IIPImage. Render hill shading by modulating pixel intensity with the dot product of the surface normals and the illumination vector, exposing the light-source incidence angle as a user-adjustable numeric parameter (entered directly or via -/+ buttons, with image updates only on button release to keep client-server exchanges reasonable). Capture the display via the snapshot mechanism as a PNG. Validate that shading varies monotonically with the incidence angle and matches the reference hill-shaded rendering of the same channel.

## Related tools

- **IIPImage (iipsrv)** (C++ FastCGI image server providing dynamic hillshading of 3D surface topologies and 32-bit floating point TIFF tile streaming via the SHD command) — https://github.com/cmarmo/iipsrv-astro
- **JavaScript** (Web-client implementation of the hillshade workflow (normal/height packing, SHD parameter updates on button release))
- **HTML5** (Standards-based client framework for the browser-rendered hillshade interface)
- **WebGL** (Interactive 3D-graphics API supporting client-side rendering of the shaded relief)

## Examples

```
curl "http://localhost/fcgi-bin/iipsrv.fcgi?FIF=dem_normals.tif&SHD=45&CVT=jpeg" -o hillshade_render.png
```

## Evaluation signals

- hillshade_render.png exists and is a valid PNG (snapshot mechanism succeeded).
- Rendered shading varies monotonically with the selected light-source incidence angle.
- Rendered output matches the reference hill-shaded rendering of the same elevation channel.
- Normal and height TIFFs are tileable and structured into a multi-resolution pyramid compatible with IIPImage streaming.
- Pixel values are derived from full-precision 32-bit elevation data, unaffected by channel mixing, image scaling, or compression.

## Limitations

- Image updates are triggered only on release of the -/+ incidence-angle buttons to keep client-server exchanges reasonable — interactive feedback is deliberately throttled.
- Server-side compression/encoding to a browser-friendly format is necessary since browser engines display gamma-encoded 8-bit-per-channel images, not 16-bit or floating point data.
- The Horn (1981) 3x3 finite-difference gradient scheme assumes a locally differentiable height surface; artifacts can arise at tile boundaries or in noisy DEMs.

## Evidence

- [readme] Dynamic hillshading of 3D surface topologies: [evidence span withheld — anchor status: external_doc]
- [readme] 32-bit floating point support: [evidence span withheld — anchor status: external_doc]
- [intro] browser gamma limits: [evidence span withheld — anchor status: not_found]
