---
name: iip-protocol-command-implementation
description: Use when serving floating-point tiled TIFF astronomy imagery to a web
  client and dynamic channel mixing, gamma, min/max cuts or pixel profiles must be
  computed from full bit-depth data rather than the browser's 8-bit display pipeline.
when_to_use_negative:
- The data is already stored as static 8-bit compressed images, where interaction
  with the pixels is essentially limited to passive visualization and server-side
  dynamic processing of full-precision values is pointless
- The processing can be done client-side on already-transcoded browser-friendly data
  (e.g. FITS parsed in-browser with jsFITS) without needing full-precision server
  values
- The target is generic document/cultural-heritage IIP serving with no astronomy-specific
  channel mixing or profile extraction needs
tools:
- name: iipsrv-astro
  role: C++ FastCGI image server in which the CMP, INV, GAM, MINMAX and PFL IIP extensions
    are implemented and which stores full-precision pixel values server-side
  repo: https://github.com/cmarmo/iipsrv-astro
- name: JavaScript
  role: Client-side language for issuing chained IIP command-value pairs in HTTP GET
    requests and rendering returned tiles
- name: HTML5
  role: Client web technology standard on which the lightweight visualization client
    that invokes the server-side commands is built
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2014.12.006
    title: Web-based visualization of very large scientific astronomy imagery
  source_content_hash: sha256:a90556e3a64f291f9e0d01475ccd36acc0bb52dfbe3ae827e1a1703eb884a84d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/cmarmo/iipsrv-astro
  tools:
  - HTML5
  - JavaScript
  - iipsrv-astro
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# iip-protocol-command-implementation

## Summary

Implement server-side IIP protocol command extensions (CMP colormap, INV inversion, GAM gamma, MINMAX per-channel cuts, PFL profile extraction) in a FastCGI image server so that astronomy pixel processing happens server-side on full-precision data, invoked by chaining command-value pairs into a single HTTP GET request from a JavaScript/HTML5 client.

## When to use

Use when serving floating-point tiled TIFF astronomy imagery to a web client and dynamic channel mixing, gamma, min/max cuts or pixel profiles must be computed from full bit-depth data rather than the browser's 8-bit display pipeline.

## When NOT to use

- The data is already stored as static 8-bit compressed images, where interaction with the pixels is essentially limited to passive visualization and server-side dynamic processing of full-precision values is pointless
- The processing can be done client-side on already-transcoded browser-friendly data (e.g. FITS parsed in-browser with jsFITS) without needing full-precision server values
- The target is generic document/cultural-heritage IIP serving with no astronomy-specific channel mixing or profile extraction needs

## Inputs

- Floating-point tiled TIFF science image (16-bit or 32-bit full precision astronomy imagery)
- HTTP GET IIP request from a JavaScript/HTML5 client with chained command-value pairs (CMP, INV, GAM, MINMAX, PFL, JTL)
- Colormap selection (e.g. JET) and gamma / min-max cut parameter values from the client's Channel Mixing panel

## Outputs

- Processed JPEG image tiles rendered under the requested CMP/INV/GAM/MINMAX settings
- profile_data JSON containing full bit-depth pixel values along lines of constant galactic longitude or latitude

## How to apply

Load the floating-point tiled TIFF science image server-side in the iipsrv-astro C++ FastCGI server, which retains full precision pixel values instead of pre-quantizing to 8 bits. Implement CMP to map pixel values through a lookup colormap (e.g. JET) offered in the client's Channel Mixing panel; implement INV (negative mode), GAM (default 2.2 for linear output on a calibrated monitor, higher to compress and lower to expand dynamic range), and per-channel minimum/maximum level cuts. Implement PFL to read full-precision pixel values directly from the scientific data — unaffected by channel mixing, scaling or compression — along lines of constant galactic longitude or latitude, serializing results as JSON. Invoke all commands by chaining command-value pairs after the image path in one HTTP GET request, e.g. combining MINMAX=b:m1,m2 and GAM=g with the tile request JTL=r,t. Only update the client image after parameter changes are finalized so server-client data exchanges remain reasonable.

## Related tools

- **iipsrv-astro** (C++ FastCGI image server in which the CMP, INV, GAM, MINMAX and PFL IIP extensions are implemented and which stores full-precision pixel values server-side) — https://github.com/cmarmo/iipsrv-astro
- **JavaScript** (Client-side language for issuing chained IIP command-value pairs in HTTP GET requests and rendering returned tiles)
- **HTML5** (Client web technology standard on which the lightweight visualization client that invokes the server-side commands is built)

## Examples

```
GET /fcgi-bin/iipsrv.fcgi?FIF=science.tif&CMP=JET&GAM=2.2&MINMAX=0:0,65535&JTL=3,0
```

## Evaluation signals

- Verify the processed tile file exists and matches a reference rendering of the same CMP/gamma/min-max settings
- Check that values in profile_data correspond to full bit-depth pixel values at the requested coordinates, unaffected by channel mixing, image scaling or compression
- Confirm a single HTTP GET request chains command-value pairs (e.g. MINMAX=b:m1,m2 with GAM=g and JTL=r,t) after the image path and is processed server-side
- Confirm gamma default is 2.2 and that increasing gamma compresses while decreasing gamma expands the displayed dynamic range
- Confirm the client image only updates after parameter changes are finalized, keeping server-client exchanges reasonable

## Limitations

- Browser engines display gamma-encoded 8-bit-per-channel images whereas scientific images require linearly quantized 16-bit or floating point values, so server-side compression and encoding remains necessary
- PFL profile values must come directly from the scientific data; any pass through channel mixing, scaling or compression would corrupt them
- The iipsrv server requires libtiff, zlib and IJG JPEG libraries (plus an fcgi-enabled web server such as Apache, Lighttpd, IIS, NginX or Tomcat), and optional Kakadu/OpenJPEG support for JPEG2000

## Evidence

- [methods] finding: [evidence span withheld — anchor status: not_found]
- [other] workflow: [evidence span withheld — anchor status: not_found]
- [readme] readme: [evidence span withheld — anchor status: external_doc]
