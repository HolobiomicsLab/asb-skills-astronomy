---
name: floating-point-to-8bit-encoding
description: Use when serving floating-point or 16-bit FITS-derived tiled TIFF science
  images to HTML5/JavaScript web clients, whose engines can only display gamma-encoded
  8-bit-per-channel images and cannot process full bit-depth pixels.
when_to_use_negative:
- Input is already a static 8-bit compressed image — interaction would be limited
  to passive visualization and no server-side dynamic processing is needed
- The client can decode FITS natively in-browser (e.g. via jsFITS) and the files are
  small enough to transfer uncompressed over the same domain
- Data are ordinary 8-bit photographs already sRGB/gamma-encoded, where server-side
  bit-depth transcoding adds no value
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: iipsrv-astro
  role: Lightweight C++ FastCGI image server that holds full-precision pixel values
    server-side and implements the CMP, INV, GAM, MINMAX and PFL IIP extensions for
    astronomy image handling
  repo: https://github.com/cmarmo/iipsrv-astro
- name: JavaScript
  role: W3C standards-based lightweight client that issues chained IIP command-value
    pairs in HTTP GET requests and renders returned 8-bit tiles
- name: HTML5
  role: Client-side standard on which the platform-independent visualization web app
    is built, compatible with touch and mouse devices
- name: VisiOmatic
  role: Web application client for visualizing astronomical images served through
    this server-side encoding pipeline
  repo: https://github.com/astromatic/visiomatic
- name: jsFITS
  role: Alternative in-browser FITS reading approach that this server-side transcoding
    skill offloads processing away from
  repo: https://github.com/slowe/jsFITS
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2014.12.006
    title: Web-based visualization of very large scientific astronomy imagery
  source_content_hash: sha256:f9b051fc97c81cea3af49f4a9a0882a73e37eff8cc63572c5e625990adc2e282
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/cmarmo/iipsrv-astro
  tools:
  - HTML5
  - JavaScript
  - VisiOmatic
  - iipsrv-astro
  - jsFITS
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# floating-point-to-8bit-encoding

## Summary

Server-side transcoding of full-precision (16-bit integer or 32-bit floating point) scientific astronomy imagery into browser-friendly 8-bit JPEG tiles, with dynamic cuts/gamma applied per request via IIP protocol extensions (CMP, INV, GAM, MINMAX, PFL) in the iipsrv-astro C++ FastCGI server.

## When to use

Use when serving floating-point or 16-bit FITS-derived tiled TIFF science images to HTML5/JavaScript web clients, whose engines can only display gamma-encoded 8-bit-per-channel images and cannot process full bit-depth pixels.

## When NOT to use

- Input is already a static 8-bit compressed image — interaction would be limited to passive visualization and no server-side dynamic processing is needed
- The client can decode FITS natively in-browser (e.g. via jsFITS) and the files are small enough to transfer uncompressed over the same domain
- Data are ordinary 8-bit photographs already sRGB/gamma-encoded, where server-side bit-depth transcoding adds no value

## Inputs

- Floating-point or 16-bit tiled TIFF science image (FITS-derived), terabyte scale
- Client IIP request with image path plus command-value pairs (CMP, GAM, MINMAX, JTL, PFL)
- Optional FILESYSTEM_PREFIX and MAX_IMAGE_CACHE_SIZE server configuration parameters

## Outputs

- 8-bit gamma-encoded JPEG image tiles streamed to the browser client
- JSON-serialized full-precision pixel profiles along lines of constant galactic longitude or latitude (profile_data)
- Dynamic JPEG export of whole image regions at any resolution

## How to apply

Store the science image server-side as a floating-point tiled TIFF so full precision pixel values never leave the server; the C++ iipsrv-astro FastCGI module decodes 1/8/16/32-bit data including 32-bit floating point. Clients chain command-value pairs after the image path in a single HTTP GET: e.g. colormap (CMP=JET from the Channel Mixing panel), inversion, gamma correction (GAM=g, default 2.2 for linear output on a calibrated monitor; raise to compress and lower to expand the dynamic range), and per-channel min/max cuts (MINMAX=b:m1,m2) combined with the tile request (JTL=r,t). All mixing, scaling and compression happen server-side because the gap between storage capacity and data access bandwidth makes it increasingly efficient to offload image processing to the server; a single server can handle more than a hundred simultaneous users accessing full precision 32-bit astronomy data. The PFL command additionally reads full-precision pixel values directly for profile extraction, serialized as JSON, unaffected by channel mixing, image scaling or compression. Update the client image only after parameter changes are finalized to keep server-client data exchanges reasonable.

## Related tools

- **iipsrv-astro** (Lightweight C++ FastCGI image server that holds full-precision pixel values server-side and implements the CMP, INV, GAM, MINMAX and PFL IIP extensions for astronomy image handling) — https://github.com/cmarmo/iipsrv-astro
- **JavaScript** (W3C standards-based lightweight client that issues chained IIP command-value pairs in HTTP GET requests and renders returned 8-bit tiles)
- **HTML5** (Client-side standard on which the platform-independent visualization web app is built, compatible with touch and mouse devices)
- **VisiOmatic** (Web application client for visualizing astronomical images served through this server-side encoding pipeline) — https://github.com/astromatic/visiomatic
- **jsFITS** (Alternative in-browser FITS reading approach that this server-side transcoding skill offloads processing away from) — https://github.com/slowe/jsFITS

## Examples

```
GET /fcgi-bin/iipsrv.fcgi?FIF=science.tif&CMP=JET&GAM=2.2&MINMAX=0:m1,m2&JTL=3,2
```

## Evaluation signals

- Processed tile file exists and matches a reference rendering produced with the same CMP colormap, GAM and MINMAX settings
- profile_data JSON values correspond to full bit-depth pixel values at the requested coordinates (unaffected by channel mixing, scaling or compression)
- Tiles decode as standard 8-bit-per-channel JPEGs loadable by a stock browser engine
- Gamma setting of 2.2 yields linear output on a properly calibrated monitor; raising GAM compresses and lowering GAM expands the dynamic range as expected
- Server sustains more than a hundred simultaneous users accessing full precision 32-bit astronomy data within MAX_IMAGE_CACHE_SIZE RAM limits

## Limitations

- Browser engines display gamma-encoded 8-bit-per-channel images, so linearly quantized 16-bit or floating-point values must be compressed and encoded server-side; full precision cannot be shown directly in the browser
- Storing data as static 8-bit compressed images limits interaction to passive visualization — dynamic settings require per-request server-side processing
- CVT dynamic JPEG export is bounded by MAX_CVT (default 5000 pixels) to prevent huge requests from overloading the server
- Parameter changes should be finalized before triggering image updates so server-client data exchanges remain reasonable

## Evidence

- [intro] server-side encoding necessity: "server-side compression and encoding of the original data to a browser-friendly format remains necessary in order to achieve a satisfactory user experience"
- [abstract] real-time full-precision access: "It provides access to floating point data at terabyte scales, with the ability to precisely adjust image settings in real-time."
- [abstract] server scalability: "a single server can comfortably handle more than a hundred simultaneous users accessing full precision 32 bit astronomy data"
- [intro] offloading rationale: [evidence span withheld — anchor status: not_found]
- [readme] iipsrv bit-depth support: [evidence span withheld — anchor status: external_doc]
