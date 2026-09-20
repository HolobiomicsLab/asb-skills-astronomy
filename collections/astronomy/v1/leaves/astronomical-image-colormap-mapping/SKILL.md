---
name: astronomical-image-colormap-mapping
description: Use when serving tiled floating-point (16/32-bit) astronomical images
  to an HTML5 web client and the client's Channel Mixing panel requests a pseudocolor
  colormap, gamma, or per-channel min/max level cuts on full-precision server-held
  pixels.
when_to_use_negative:
- Input is already an 8-bit browser-encoded image — the whole point is applying colormap/cuts
  to full-precision server-side data, since browser engines only display gamma-encoded
  8-bit-per-channel images
- The analysis requires raw numeric pixel values rather than visualization — use the
  profile-extraction command instead, which reads full-precision values unaffected
  by channel mixing, scaling or compression
- Data are not tiled/scientific imagery (e.g. plain static 8-bit compressed images),
  where interaction is limited to passive visualization and server-side dynamic processing
  adds no value
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: iipsrv-astro (IIPImage server)
  role: C++ FastCGI image server that stores full-precision pixel values and performs
    the server-side CMP colormap, inversion, GAM gamma, and MINMAX level-cut processing
    before encoding tiles
  repo: https://github.com/cmarmo/iipsrv-astro
- name: JavaScript/HTML5 client
  role: Lightweight web client whose Channel Mixing panel issues the colormap/gamma/min-max
    parameters chained into the IIP HTTP GET request and renders the returned tiles
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ascom.2014.12.006
    title: Web-based visualization of very large scientific astronomy imagery
  source_content_hash: sha256:41e1089779e8edfb9bc7dcc3ebda83477f33c9a3162ac7dde2e25bda26540085
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/cmarmo/iipsrv-astro
  tools:
  - JavaScript/HTML5 client
  - iipsrv-astro (IIPImage server)
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# astronomical-image-colormap-mapping

## Summary

Server-side colormap mapping of full-precision floating-point astronomy imagery, where the IIP server (iipsrv-astro) applies a lookup colormap (e.g. JET), gamma correction, and per-channel min/max cuts before encoding browser tiles — avoiding any lossy client-side rescaling of 32-bit float pixel data.

## When to use

Use when serving tiled floating-point (16/32-bit) astronomical images to an HTML5 web client and the client's Channel Mixing panel requests a pseudocolor colormap, gamma, or per-channel min/max level cuts on full-precision server-held pixels.

## When NOT to use

- Input is already an 8-bit browser-encoded image — the whole point is applying colormap/cuts to full-precision server-side data, since browser engines only display gamma-encoded 8-bit-per-channel images
- The analysis requires raw numeric pixel values rather than visualization — use the profile-extraction command instead, which reads full-precision values unaffected by channel mixing, scaling or compression
- Data are not tiled/scientific imagery (e.g. plain static 8-bit compressed images), where interaction is limited to passive visualization and server-side dynamic processing adds no value

## Inputs

- Floating-point tiled TIFF astronomy image (32-bit float per channel) hosted server-side by iipsrv-astro
- Client parameters from the Channel Mixing panel: colormap name (e.g. JET), gamma value (default 2.2), inversion flag, per-channel min/max cut values
- Tile coordinates (resolution, tile index) from the HTML5/JavaScript client

## Outputs

- Colormap-mapped, gamma-corrected, min/max-cut 8-bit compressed image tiles (e.g. JPEG) served via the IIP protocol
- Updated client visualization in the HTML5/JavaScript web app after parameter finalization

## How to apply

Load the floating-point tiled TIFF science image into the iipsrv-astro C++ FastCGI server, which retains full-precision pixel values server-side so colormap mapping never quantizes the source data. Implement the CMP command so that pixel values are mapped through a named lookup colormap (e.g. JET) selected from the color maps offered in the client's Channel Mixing panel. Couple it with the inversion (negative) mode, gamma correction (default 2.2 for linear output on a properly calibrated monitor; raise it to compress and lower it to expand the dynamic range), and per-channel minimum/maximum level cuts (MINMAX=b:m1,m2). Chain these command-value pairs with the tile request (JTL=r,t) after the image path in a single HTTP GET so all processing happens server-side, and return the processed tiles to the JavaScript/HTML5 client — refreshing the image only after parameter changes are finalized to keep server-client exchanges reasonable.

## Related tools

- **iipsrv-astro (IIPImage server)** (C++ FastCGI image server that stores full-precision pixel values and performs the server-side CMP colormap, inversion, GAM gamma, and MINMAX level-cut processing before encoding tiles) — https://github.com/cmarmo/iipsrv-astro
- **JavaScript/HTML5 client** (Lightweight web client whose Channel Mixing panel issues the colormap/gamma/min-max parameters chained into the IIP HTTP GET request and renders the returned tiles)

## Examples

```
GET /fcgi-bin/iipsrv.fcgi?FIF=science_image.tif&CMP=JET&INV&GAM=2.2&MINMAX=0:0.001,0.999&JTL=3,10
```

## Evaluation signals

- Verify the processed tile file exists and matches a reference rendering produced with the same CMP colormap, gamma, and min-max settings
- Confirm the request URL chains all commands after the image path in one HTTP GET (e.g. FIF=...&CMP=JET&GAM=2.2&MINMAX=...&JTL=r,t) and all are handled server-side
- Check that changing the colormap name (e.g. JET) in the Channel Mixing panel changes the returned tile palette while the underlying 32-bit float data remain untouched
- Confirm the image refreshes only after parameter changes are finalized, keeping server-client data exchanges within reasonable limits
- Verify gamma defaults to 2.2 for linear output and that raising/lowering gamma visibly compresses/expands the dynamic range as specified

## Limitations

- Browser engines only display gamma-encoded 8-bit-per-channel images, so server-side compression and encoding of the original linearly quantized 16-bit/floating-point data is mandatory — colormap output is inherently an 8-bit rendering
- Image updates should be deferred until parameter changes are finalized to keep server-client data exchanges reasonable; interactive per-parameter streaming is not the design goal
- Gamma correctness depends on a properly calibrated monitor; the 2.2 default yields linear output only under that assumption

## Evidence

- [other] CMP colormap command mapping: [evidence span withheld — anchor status: not_found]
- [other] gamma default and dynamic range: [evidence span withheld — anchor status: not_found]
- [other] IIP command chaining: [evidence span withheld — anchor status: not_found]
- [readme] server full-precision support: [evidence span withheld — anchor status: external_doc]
