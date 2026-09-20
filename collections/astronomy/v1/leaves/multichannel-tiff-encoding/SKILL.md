---
name: multichannel-tiff-encoding
description: Use when derived per-pixel scientific data (vector fields, height maps,
  normal components) must be packaged for streaming visualization via IIPImage-style
  tiled servers, or when full 32-bit floating point precision must survive channel
  mixing and compression.
when_to_use_negative:
- Input imagery is display-oriented 8-bit RGB already gamma-encoded for browsers —
  the browser engines display gamma-encoded 8-bit images, so no scientific re-encoding
  value is added
- The goal is passive visualization of a static image; storing data as static 8-bit
  compressed images limits interaction to passive visualization, but there is no dynamic
  channel (normals/heights) to encode
- The source raster has no elevation or surface-gradient semantics — Horn (1981) gradient
  estimation and normal packing would be meaningless
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_3372
tools:
- name: IIPImage (iipsrv)
  role: Server that streams the tiled multi-resolution 32-bit TIFFs and performs dynamic
    hillshading of 3D surface topologies via the SHD command
  repo: https://github.com/cmarmo/iipsrv-astro
- name: VisiOmatic
  role: JavaScript/HTML5 web client that queries compressed tiles from the IIPImage
    server for browser-side visualization
  repo: https://github.com/astromatic/visiomatic
- name: RTWeb3D / Mizar
  role: WebGL-based 3D browser application for visualizing geospatial data for sky
    and planets, consuming streamed geospatial protocols
  repo: https://github.com/TPZF/RTWeb3D
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.ascom.2014.12.006
    title: Web-based visualization of very large scientific astronomy imagery
  source_content_hash: sha256:16ec1ce8df04c5d01853bb9ba1858374f98ee73b9582831a5ba6a5be19591c41
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/cmarmo/iipsrv-astro
  tools:
  - IIPImage (iipsrv)
  - RTWeb3D / Mizar
  - VisiOmatic
  - WebGL
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_3372
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# multichannel-tiff-encoding

## Summary

Encodes derived scientific channels (e.g. per-pixel surface normals and heights for hillshading) into standard, tileable multi-resolution 32-bit TIFFs — normal XYZ vectors packed into a 3-channel 'color' TIFF and height into a separate 1-channel monochrome TIFF — so an IIPImage server can stream them to lightweight web clients.

## When to use

Use when derived per-pixel scientific data (vector fields, height maps, normal components) must be packaged for streaming visualization via IIPImage-style tiled servers, or when full 32-bit floating point precision must survive channel mixing and compression.

## When NOT to use

- Input imagery is display-oriented 8-bit RGB already gamma-encoded for browsers — the browser engines display gamma-encoded 8-bit images, so no scientific re-encoding value is added
- The goal is passive visualization of a static image; storing data as static 8-bit compressed images limits interaction to passive visualization, but there is no dynamic channel (normals/heights) to encode
- The source raster has no elevation or surface-gradient semantics — Horn (1981) gradient estimation and normal packing would be meaningless

## Inputs

- 32-bit floating point elevation/height channel raster (DEM point data)
- Tiled multi-resolution image pyramid structure suitable for IIPImage streaming

## Outputs

- 3-channel 32-bit TIFF ('color') containing per-pixel XYZ surface normal vectors
- 1-channel monochrome 32-bit TIFF containing per-pixel height values
- Rendered hill-shaded relief image (e.g. hillshade_render.png snapshot)

## How to apply

Load the full-precision (32-bit) pixel values of the source channel, which are unaffected by client-side channel mixing, image scaling or compression. Estimate per-pixel surface gradients from the height map over a 3x3 neighborhood using the Horn (1981) finite-difference scheme, then derive the surface normal components and height per pixel. Pack the XYZ normal components into the three channels of a tileable 32-bit TIFF and the height values into a separate single-channel monochrome TIFF, structuring both as multi-resolution pyramids so the IIPImage server can stream them. Downstream, hill-shaded relief is rendered by modulating pixel intensity with the dot product of surface normals and the illumination vector, with the light-source incidence angle as a user-adjustable parameter handled by the IIP SHD command. Keep the pipeline at full bit depth end-to-end, as the IIP SHD command does, so browser-side 8-bit gamma encoding does not degrade the derived data.

## Related tools

- **IIPImage (iipsrv)** (Server that streams the tiled multi-resolution 32-bit TIFFs and performs dynamic hillshading of 3D surface topologies via the SHD command) — https://github.com/cmarmo/iipsrv-astro
- **VisiOmatic** (JavaScript/HTML5 web client that queries compressed tiles from the IIPImage server for browser-side visualization) — https://github.com/astromatic/visiomatic
- **RTWeb3D / Mizar** (WebGL-based 3D browser application for visualizing geospatial data for sky and planets, consuming streamed geospatial protocols) — https://github.com/TPZF/RTWeb3D

## Examples

```
iipsrv.fcgi?FIF=dem_normal.tif&SHD=90&CVT=jpeg
```

## Evaluation signals

- hillshade_render.png exists and is a valid PNG snapshot of the rendered hill-shaded display
- Rendered shading varies monotonically with the selected light-source incidence angle
- Output matches the reference hill-shaded rendering of the same channel
- Output TIFFs are tileable and structured into a multi-resolution pyramid for IIPImage streaming
- Channel bit depth is 32-bit full precision, unaffected by channel mixing, image scaling or compression

## Limitations

- Client-server exchanges must be kept reasonable: the hillshade image updates only when the incidence-angle buttons (-/+) are released, not continuously
- Browser engines display gamma-encoded 8-bit-per-channel images, so server-side compression and encoding remain necessary; precision must be preserved server-side, not client-side
- The server (iipsrv) requires libtiff, zlib and IJG JPEG libraries, and the image cache is bounded by MAX_IMAGE_CACHE_SIZE (default 10MB), constraining streaming volume

## Evidence

- [supplementary] XYZ normal vectors are packed into a 3-channel 'color' TIFF and the height data into a separate 1-channel monochrome TIFF, both tileable and structured into a multi-resolution pyramid for streaming with IIPImage.: [evidence span withheld — anchor status: not_found]
- [supplementary] Estimate per-pixel surface gradients from the height map over a 3x3 neighborhood using the Horn (1981) finite-difference scheme: [evidence span withheld — anchor status: not_found]
- [readme] Dynamic hillshading of 3D surface topologies (README feature list): [evidence span withheld — anchor status: external_doc]
- [readme] 1, 8, 16 and 32 bit image support including 32 bit floating point support: [evidence span withheld — anchor status: external_doc]
