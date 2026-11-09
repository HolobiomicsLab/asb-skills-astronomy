---
name: server-side-image-processing-pipeline
description: Use when serving very large (terabyte-scale) floating-point astronomy
  images to web clients and you need real-time image setting adjustments without shipping
  full bit-depth data to the browser.
when_to_use_negative:
- The data are already static 8-bit compressed browser-friendly images — interaction
  would be limited to passive visualization and the server-side precision pipeline
  adds no value.
- The goal is client-side decoding of FITS directly in the browser (e.g. via jsFITS)
  rather than server-side streaming of processed tiles.
- Input images are small or low-dynamic-range enough that transcoding/processing within
  the browser using JavaScript is acceptable.
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: iipsrv-astro
  role: C++ FastCGI image server that stores full-precision pixel values and implements
    the astronomy IIP extensions (CMP, INV, GAM, MINMAX, PFL) server-side
  repo: https://github.com/cmarmo/iipsrv-astro
- name: JavaScript
  role: W3C standards-based light-weight client that issues chained IIP command-value
    requests and renders the returned tiles
- name: HTML5
  role: Standard web technology on which the platform-independent touch/mouse-compatible
    visualization client is built
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2014.12.006
    title: Web-based visualization of very large scientific astronomy imagery
  source_content_hash: sha256:ff68ab4a30218666b16bf77273ec313d75b599a0f11499ab86f69985b2db112d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/cmarmo/iipsrv-astro
  tools:
  - HTML5
  - JavaScript
  - iipsrv-astro
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# server-side-image-processing-pipeline

## Summary

Offload astronomy image processing (colormap mapping, inversion, gamma correction, per-channel min/max cuts, profile extraction) to a lightweight C++ FastCGI server (iipsrv-astro) that reads full-precision 32-bit floating-point tiled TIFF data, instead of transcoding data in the browser.

## When to use

Use when serving very large (terabyte-scale) floating-point astronomy images to web clients and you need real-time image setting adjustments without shipping full bit-depth data to the browser.

## When NOT to use

- The data are already static 8-bit compressed browser-friendly images — interaction would be limited to passive visualization and the server-side precision pipeline adds no value.
- The goal is client-side decoding of FITS directly in the browser (e.g. via jsFITS) rather than server-side streaming of processed tiles.
- Input images are small or low-dynamic-range enough that transcoding/processing within the browser using JavaScript is acceptable.

## Inputs

- floating-point (32-bit) tiled TIFF astronomy science images
- client HTTP GET requests with chained IIP command-value pairs (JTL, CMP, GAM, MINMAX, inversion, PFL)

## Outputs

- processed JPEG image tiles rendered with the requested colormap, gamma and min/max cuts
- JSON-serialized pixel profiles at full bit depth along constant galactic longitude/latitude lines

## How to apply

Load the floating-point tiled TIFF science image into the iipsrv-astro C++ FastCGI server, which stores full-precision pixel values server-side. Implement the new IIP protocol extensions as command-value pairs chained after the image path in a single HTTP GET request: CMP for colormap lookup (e.g. JET), gamma correction (GAM=g, default 2.2 for linear output on a calibrated monitor), per-channel minimum/maximum level cuts (MINMAX=b:m1,m2), and inversion (negative mode), combined with tile requests such as JTL=r,t, all processed server-side. Implement a profile-extraction command that reads full-precision pixel values directly from the scientific data — unaffected by channel mixing, image scaling or compression — along lines of constant galactic longitude or latitude, and serialize them as JSON. Update the image in the JavaScript/HTML5 client only after parameter changes are finalized to keep server-client data exchanges reasonable. Rationale: browser engines display gamma-encoded 8-bit-per-channel images whereas scientific images require 16-bit or floating-point values, so server-side compression and encoding is necessary.

## Related tools

- **iipsrv-astro** (C++ FastCGI image server that stores full-precision pixel values and implements the astronomy IIP extensions (CMP, INV, GAM, MINMAX, PFL) server-side) — https://github.com/cmarmo/iipsrv-astro
- **JavaScript** (W3C standards-based light-weight client that issues chained IIP command-value requests and renders the returned tiles)
- **HTML5** (Standard web technology on which the platform-independent touch/mouse-compatible visualization client is built)

## Examples

```
GET /fcgi-bin/iipsrv.fcgi?FIF=image.tif&JTL=5,3&MINMAX=0:100,2000&GAM=2.2&CMP=JET
```

## Evaluation signals

- Verify the processed tile file exists and matches a reference rendering produced with the same CMP colormap, gamma and min-max cut settings.
- Check that values in profile_data correspond to full bit-depth pixel values at the requested coordinates (unaffected by channel mixing, scaling or compression).
- Confirm the server handles 8, 16 and 32 bits per channel including 32-bit floating point support as declared in the iipsrv README.
- Confirm a single server can comfortably handle more than a hundred simultaneous users accessing full precision 32-bit astronomy data.
- Verify image updates in the client occur only after parameter changes are finalized, keeping server-client data exchanges reasonable.

## Limitations

- Server-side compression and encoding of the original data to a browser-friendly format remains necessary because browser engines display gamma-encoded 8-bit-per-channel images.
- Gamma default of 2.2 assumes a properly calibrated monitor; higher values compress and lower values expand the dynamic range.
- JPEG2000 decoding via OpenJPEG is experimental and much slower than Kakadu, and is disabled by default in iipsrv.
- The client must avoid updating the image on every intermediate parameter change to keep server-client data exchanges reasonable.

## Evidence

- [full_text] The new IIP extensions are invoked by chaining command-value pairs after the image path in a single HTTP GET request: [evidence span withheld — anchor status: not_found]
- [intro] static 8-bit limitation: [evidence span withheld — anchor status: not_found]
- [readme] 32-bit floating point support: [evidence span withheld — anchor status: external_doc]
- [readme] openjpeg slower than kakadu: [evidence span withheld — anchor status: external_doc]
