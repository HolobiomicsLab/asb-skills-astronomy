---
name: profile-pixel-extraction-serialization
description: Use when a client needs a pixel-intensity profile (e.g. along lines of
  constant galactic longitude or latitude) from full bit-depth FITS/tiled-TIFF astronomy
  data hosted server-side, and the profile must reflect raw science values rather
  than the gamma/mixing-adjusted display.
when_to_use_negative:
- Data are stored as static 8-bit compressed images with no server-side full-precision
  access — interaction is then limited to passive visualization and raw pixel profiles
  cannot be extracted
- The client needs the profile of the rendered/appearance-modified image (colormap,
  gamma, min-max cuts applied) — PFL reads values unaffected by channel mixing, scaling
  or compression by design
- The source data must be decoded and processed in the browser (e.g. via jsFITS) rather
  than offloaded server-side, which the article argues is less efficient given the
  growing gap between storage capacity and data access bandwidth
tools:
- name: iipsrv-astro
  role: C++ FastCGI image server implementing the IIP astronomy extensions (CMP, INV,
    GAM, MINMAX, PFL); reads full-precision pixels from the tiled TIFF science data
    and serializes profiles as JSON
  repo: https://github.com/cmarmo/iipsrv-astro
- name: JavaScript
  role: Client-side language that issues the chained HTTP GET IIP request and receives/consumes
    the JSON profile data in the HTML5 web app
- name: HTML5
  role: Standard web technology on which the lightweight platform-independent client
    displaying the profile is built
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2014.12.006
    title: Web-based visualization of very large scientific astronomy imagery
  source_content_hash: sha256:0164dd7e2a3fd90e86fe6be802544f3a977ced5483479fa54db8b3566945bcda
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
yes_rate: 0.6
n_claims: 5
---

# profile-pixel-extraction--serialization

## Summary

Extract intensity profiles from full-precision (floating-point / 32-bit) pixels of tiled scientific astronomy imagery served by the iipsrv-astro C++ server via the IIP profile command (PFL), reading values directly from the science data so they are unaffected by channel mixing, image scaling or compression, and serialize the result as JSON for the web client.

## When to use

Use when a client needs a pixel-intensity profile (e.g. along lines of constant galactic longitude or latitude) from full bit-depth FITS/tiled-TIFF astronomy data hosted server-side, and the profile must reflect raw science values rather than the gamma/mixing-adjusted display.

## When NOT to use

- Data are stored as static 8-bit compressed images with no server-side full-precision access — interaction is then limited to passive visualization and raw pixel profiles cannot be extracted
- The client needs the profile of the rendered/appearance-modified image (colormap, gamma, min-max cuts applied) — PFL reads values unaffected by channel mixing, scaling or compression by design
- The source data must be decoded and processed in the browser (e.g. via jsFITS) rather than offloaded server-side, which the article argues is less efficient given the growing gap between storage capacity and data access bandwidth

## Inputs

- Floating-point tiled TIFF science image (e.g. FITS-derived 32-bit astronomy imagery) served by iipsrv-astro
- HTTP GET IIP request chaining the image path with command-value pairs (FIF, PFL, optionally CMP/GAM/MINMAX/JTL)

## Outputs

- JSON-serialized profile of full bit-depth pixel values along the requested line of constant galactic longitude or latitude (profile_data)
- JSON metadata returned to the JavaScript/HTML5 client together with processed image tiles

## How to apply

Serve the floating-point tiled TIFF science image with the iipsrv-astro FastCGI server, which stores full-precision pixel values server-side (it supports 1, 8, 16 and 32 bit channels including 32-bit floating point). The client invokes the profile command by chaining command-value pairs after the image path in a single HTTP GET request, alongside other astronomy extensions such as CMP (colormap, e.g. JET), GAM (gamma, default 2.2 for linear output on a calibrated monitor), MINMAX (per-channel min/max cuts) and JTL (tile request). The server reads full-precision pixel values directly from the scientific data — explicitly bypassing any channel mixing, image scaling or JPEG compression — along the requested line of constant galactic longitude or latitude, and serializes them as JSON metadata returned to the JavaScript/HTML5 client. To keep server-client data exchanges reasonable, the image is only refreshed after parameter changes are finalized, not on every intermediate adjustment. A single server can handle over a hundred simultaneous users on full precision 32-bit data, so profiles can be extracted interactively without precomputing.

## Related tools

- **iipsrv-astro** (C++ FastCGI image server implementing the IIP astronomy extensions (CMP, INV, GAM, MINMAX, PFL); reads full-precision pixels from the tiled TIFF science data and serializes profiles as JSON) — https://github.com/cmarmo/iipsrv-astro
- **JavaScript** (Client-side language that issues the chained HTTP GET IIP request and receives/consumes the JSON profile data in the HTML5 web app)
- **HTML5** (Standard web technology on which the lightweight platform-independent client displaying the profile is built)

## Examples

```
curl "http://host/fcgi-bin/iipsrv.fcgi?FIF=science_image.tif&PFL=<galLon>,<galLat>"  # chained in one HTTP GET, e.g. with &CMP=JET&GAM=2.2&MINMAX=b:m1,m2&JTL=r,t
```

## Evaluation signals

- The returned profile_data JSON parses and contains values corresponding to full bit-depth pixel values at the requested coordinates
- Profile values are invariant under changes to CMP colormap, GAM gamma and MINMAX min/max-cut settings, confirming extraction is unaffected by channel mixing, scaling or compression
- The processed tile file exists and matches a reference rendering of the same CMP/gamma/min-max settings used in the same request chain
- Extracted profile values fall within the dynamic range of the source floating-point science data (full precision 32-bit), not the 8-bit gamma-encoded browser range
- Updating the profile only occurs after parameter changes are finalized, keeping server-client data exchanges reasonable

## Limitations

- Bypasses display-side processing: profile values will not match what the user sees after gamma (default 2.2), colormap or min-max adjustments
- Requires the server to hold full-precision data (32-bit floating point support); browser engines natively display only gamma-encoded 8-bit-per-channel images, so server-side processing is mandatory
- Profile extraction is restricted to lines of constant galactic longitude or latitude in the astronomy use case
- No changelog found for the underlying repositories, making version-dependent behavior of the extension commands hard to track

## Evidence

- [methods] profile extraction reads full-precision pixels: [evidence span withheld — anchor status: not_found]
- [methods] profiles serialize as JSON: [evidence span withheld — anchor status: not_found]
- [readme] 32-bit floating point server support: [evidence span withheld — anchor status: external_doc]
- [intro] server-side offload rationale: [evidence span withheld — anchor status: not_found]
