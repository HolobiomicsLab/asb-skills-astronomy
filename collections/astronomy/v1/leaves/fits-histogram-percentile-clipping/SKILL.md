---
name: fits-histogram-percentile-clipping
description: Use when transcoding FITS science data to a browser-friendly TIFF format
  and the output must carry display clipping metadata (SMinSampleValue/SMaxSampleValue)
  computed from the image histogram rather than hard-coded min/max pixel values.
when_to_use_negative:
- The input is not a FITS science image with a meaningful pixel-value histogram (e.g.
  an already 8-bit compressed display image), where percentile clipping of the original
  data would be meaningless.
- You need dynamic real-time adjustment of image settings in the client; the article's
  system instead supports real-time setting adjustment on full-precision floating-point
  data server-side rather than fixed tags baked into an 8-bit render.
- Display bounds are already provided externally (e.g. by the user or upstream scale_parameters),
  so recomputing Smin/Smax from the histogram would override deliberate values.
tools:
- name: JavaScript
  role: Runtime of the W3C-standards-based clients that consume the transcoded, clipped
    imagery served for web visualization
- name: iipsrv-astro (IIPImage server)
  role: High-resolution streaming image server that reads the transcoded TIFF images
    (including SMinSampleValue/SMaxSampleValue tags) and serves tiles to web clients
  repo: https://github.com/cmarmo/iipsrv-astro
- name: jsFITS
  role: JavaScript FITS library for reading FITS image data directly in the browser,
    an alternative to server-side transcoding of the same FITS input
  repo: https://github.com/slowe/jsFITS
- name: VisiOmatic
  role: Web visualization client that consumes server-streamed astronomical imagery
    for display with adjustable settings
  repo: https://github.com/astromatic/visiomatic
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2014.12.006
    title: Web-based visualization of very large scientific astronomy imagery
  source_content_hash: sha256:f9673a059b6e31a91b4390d5320659a986cc374308104c76e86455684279423c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/cmarmo/iipsrv-astro
  tools:
  - JavaScript
  - VisiOmatic
  - iipsrv-astro (IIPImage server)
  - jsFITS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# fits-histogram-percentile-clipping

## Summary

Reconstructs the STIFF display-scale computation that derives Smin/Smax metadata from a FITS science image histogram, so that browser-friendly transcoded tiles (e.g. for the IIPImage server) carry meaningful intensity clipping bounds. Smax is the 99.9th percentile of the pixel histogram and Smin places the median sky level at a fixed low fraction of the display range.

## When to use

Use when transcoding FITS science data to a browser-friendly TIFF format and the output must carry display clipping metadata (SMinSampleValue/SMaxSampleValue) computed from the image histogram rather than hard-coded min/max pixel values.

## When NOT to use

- The input is not a FITS science image with a meaningful pixel-value histogram (e.g. an already 8-bit compressed display image), where percentile clipping of the original data would be meaningless.
- You need dynamic real-time adjustment of image settings in the client; the article's system instead supports real-time setting adjustment on full-precision floating-point data server-side rather than fixed tags baked into an 8-bit render.
- Display bounds are already provided externally (e.g. by the user or upstream scale_parameters), so recomputing Smin/Smax from the histogram would override deliberate values.

## Inputs

- FITS science image pixel array (e.g. read with astropy.io.fits)
- Fixed clipping ratio rho_sky = 0.001
- Percentile threshold: 999th permil (99.9th percentile) for Smax

## Outputs

- Smax (99.9th percentile of the pixel-value histogram)
- Ssky (median of all pixel values)
- Smin computed as (Ssky - rho_sky * Smax) / (1 - rho_sky)
- TIFF tags SMinSampleValue and SMaxSampleValue written into the transcoded image
- scale_parameters export containing the (Smin, Ssky, Smax) triplet

## How to apply

Load the FITS science image pixel array (the same input consumed by the transcoding step) with astropy.io.fits and accumulate the full pixel-value histogram. Set the upper display bound Smax to the 999th permil (99.9th percentile) of that histogram, following the STIFF default. Estimate the sky level Ssky as the median of all pixel values, then derive the lower display bound via Smin = (Ssky - rho_sky * Smax) / (1 - rho_sky) with the fixed ratio rho_sky = 0.001, so the sky background appears at about 0.001 of the maximum display radiant emittance. Write Smin and Smax into the SMinSampleValue and SMaxSampleValue TIFF tags of the transcoded, browser-friendly image, and export the triplet (Smin, Ssky, Smax) to scale_parameters for downstream clients.

## Related tools

- **JavaScript** (Runtime of the W3C-standards-based clients that consume the transcoded, clipped imagery served for web visualization)
- **iipsrv-astro (IIPImage server)** (High-resolution streaming image server that reads the transcoded TIFF images (including SMinSampleValue/SMaxSampleValue tags) and serves tiles to web clients) — https://github.com/cmarmo/iipsrv-astro
- **jsFITS** (JavaScript FITS library for reading FITS image data directly in the browser, an alternative to server-side transcoding of the same FITS input) — https://github.com/slowe/jsFITS
- **VisiOmatic** (Web visualization client that consumes server-streamed astronomical imagery for display with adjustable settings) — https://github.com/astromatic/visiomatic

## Examples

```
from astropy.io import fits; import numpy as np; d = fits.getdata('science.fits').ravel(); smax = np.percentile(d, 99.9); ssky = np.median(d); smin = (ssky - 0.001 * smax) / (1 - 0.001)
```

## Evaluation signals

- Invariant check: Smin < Ssky < Smax must hold for the computed triplet.
- Smax must equal the 99.9th percentile (999th permil) of the accumulated FITS pixel-value histogram.
- Ssky must equal the median of all pixel values, and Smin must satisfy Smin = (Ssky - 0.001 * Smax) / (1 - 0.001) exactly.
- The transcoded TIFF's SMinSampleValue and SMaxSampleValue tags must contain exactly the computed Smin and Smax.
- scale_parameters must contain the full triplet (Smin, Ssky, Smax) consistent with the TIFF tags.

## Limitations

- The sky-background model assumes the median pixel value represents the sky; images with extended sources or unusual backgrounds may violate this assumption.
- The fixed rho_sky = 0.001 and the 99.9th-percentile Smax are defaults; different display requirements would need different thresholds.
- Browser engines display gamma-encoded 8-bit-per-channel images, so clipping metadata is only meaningful in the server-side transcoding path, not for raw 16-bit or floating-point FITS values in the browser.
- Storing data as static 8-bit compressed images limits interaction to passive visualization; the fixed clip bounds cannot reproduce the real-time setting adjustment available on full-precision server-side data.

## Evidence

- [other] STIFF Smin/Smax defaults: [evidence span withheld — anchor status: not_found]
- [other] Smin/Smax workflow: [evidence span withheld — anchor status: not_found]
- [other] TIFF tag targets: [evidence span withheld — anchor status: not_found]
- [intro] server-side transcoding rationale: "server-side compression and encoding of the original data to a browser-friendly format remains necessary in order to achieve a satisfactory user experience"
- [abstract] real-time settings on full-precision data: "It provides access to floating point data at terabyte scales, with the ability to precisely adjust image settings in real-time."
