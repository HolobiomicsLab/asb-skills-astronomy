---
name: astronomical-sky-background-median-estimation
description: Use when transcoding FITS science imagery to a browser-friendly TIFF
  for IIPImage/VisiOmatic and default Smin/Smax display cuts must be derived from
  the pixel-value histogram and the sky median.
when_to_use_negative:
- The transcoded image already carries curated SMinSampleValue/SMaxSampleValue cuts
  that should be preserved rather than overwritten with histogram-derived defaults.
- Only already-rendered 8-bit PNG/JPEG tiles or header/catalog data are available,
  so no full pixel-value histogram or median can be accumulated.
- Science data is converted to images directly within the browser from FITS (e.g.,
  jsFITS-style client-side decoding), bypassing server-side TIFF display-scale metadata.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: STIFF
  role: Reference transcoding tool whose default Smin/Smax display-scale computation
    this skill reconstructs and whose SMinSampleValue/SMaxSampleValue TIFF tags it
    populates
- name: astropy
  role: astropy.io.fits loads the FITS science pixel array from which the histogram,
    the 99.9th-percentile Smax, and the median Ssky are computed
- name: JavaScript
  role: Language of the W3C standards-based browser clients that display the imagery
    scaled by the computed Smin/Smax values
- name: iipsrv (IIPImage server)
  role: Serves the transcoded 16/32-bit scientific TIFF imagery to browsers, for which
    the STIFF display scale information is stored via the SMinSampleValue/SMaxSampleValue
    tags
  repo: https://github.com/cmarmo/iipsrv-astro
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2014.12.006
    title: Web-based visualization of very large scientific astronomy imagery
  source_content_hash: sha256:e513f17043f52b7bc12dc4a0a0ffaf291ed9df3f9ccccd99b10b806960651557
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/cmarmo/iipsrv-astro
  tools:
  - JavaScript
  - STIFF
  - astropy
  - iipsrv (IIPImage server)
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# astronomical-sky-background-median-estimation

## Summary

Reconstructs the STIFF display-scale metadata computed when transcoding full bit-depth FITS science images to a browser-friendly TIFF for the IIPImage server: Smax is the 999th permil of the pixel-value histogram, Ssky is the median of all pixels, and Smin places the sky background at rho_sky = 0.001 of the maximum display radiant emittance.

## When to use

Use when transcoding FITS science imagery to a browser-friendly TIFF for IIPImage/VisiOmatic and default Smin/Smax display cuts must be derived from the pixel-value histogram and the sky median.

## When NOT to use

- The transcoded image already carries curated SMinSampleValue/SMaxSampleValue cuts that should be preserved rather than overwritten with histogram-derived defaults.
- Only already-rendered 8-bit PNG/JPEG tiles or header/catalog data are available, so no full pixel-value histogram or median can be accumulated.
- Science data is converted to images directly within the browser from FITS (e.g., jsFITS-style client-side decoding), bypassing server-side TIFF display-scale metadata.

## Inputs

- FITS science image pixel array (loaded via astropy.io.fits)
- Fixed display ratio rho_sky = 0.001 (STIFF default)
- Target browser-friendly transcoded TIFF image that will carry the sample-value tags

## Outputs

- Smax: 999th permil (99.9th percentile) of the pixel-value histogram
- Ssky: median of all pixel values (sky background level)
- Smin: (Ssky - rho_sky * Smax) / (1 - rho_sky)
- SMinSampleValue and SMaxSampleValue TIFF tags written on the transcoded image
- (Smin, Ssky, Smax) triplet exported to scale_parameters

## How to apply

Load the FITS science image pixel array (the same input consumed by the transcoding step) with astropy.io.fits and accumulate the full pixel-value histogram, then set the upper display bound Smax to the 999th permil (99.9th percentile) of that histogram and estimate the sky level Ssky as the median of all pixel values. Derive the lower bound as Smin = (Ssky - rho_sky * Smax) / (1 - rho_sky) with the fixed ratio rho_sky = 0.001, which renders the sky background at about 0.001 of the maximum display radiant emittance (essentially black). Write Smin and Smax into the SMinSampleValue and SMaxSampleValue TIFF tags of the browser-friendly transcoded image; this is needed because browser engines only display gamma-encoded 8-bit-per-channel images while the IIPImage server streams linearly quantized 16-bit or floating-point scientific data. Finally, export the full triplet (Smin, Ssky, Smax) to scale_parameters so the display cuts remain auditable and adjustable in real time.

## Related tools

- **STIFF** (Reference transcoding tool whose default Smin/Smax display-scale computation this skill reconstructs and whose SMinSampleValue/SMaxSampleValue TIFF tags it populates)
- **astropy** (astropy.io.fits loads the FITS science pixel array from which the histogram, the 99.9th-percentile Smax, and the median Ssky are computed)
- **JavaScript** (Language of the W3C standards-based browser clients that display the imagery scaled by the computed Smin/Smax values)
- **iipsrv (IIPImage server)** (Serves the transcoded 16/32-bit scientific TIFF imagery to browsers, for which the STIFF display scale information is stored via the SMinSampleValue/SMaxSampleValue tags) — https://github.com/cmarmo/iipsrv-astro

## Examples

```
import numpy as np; from astropy.io import fits; d = fits.getdata('science.fits'); smax = np.percentile(d, 99.9); ssky = np.median(d); smin = (ssky - 0.001*smax)/(1 - 0.001)
```

## Evaluation signals

- Ordering invariant holds: Smin < Ssky < Smax for the exported triplet.
- SMaxSampleValue TIFF tag contains exactly the computed 99.9th-percentile Smax.
- SMinSampleValue TIFF tag contains exactly Smin = (Ssky - 0.001 * Smax) / (1 - 0.001).
- Rerunning on the same FITS input reproduces identical tag values (deterministic histogram and median, no stochastic steps).
- scale_parameters export carries the complete (Smin, Ssky, Smax) triplet, not just the two tag values.

## Limitations

- The cuts are fixed STIFF defaults (999th permil for Smax, rho_sky = 0.001), not an adaptive fit; images needing different rendering require manual overrides.
- Browser engines display gamma-encoded 8-bit-per-channel images whereas scientific images are linearly quantized 16-bit or floating point, so the derived tags only approximate the intended on-screen mapping.
- Median-based Ssky presumes a dominant sky background in the pixel distribution; no fallback is defined for images where sources fill most of the frame.

## Evidence

- [other] STIFF sets Smax to the 999th permil of the image histogram by default, and computes Smin such that the sky background Ssky (taken as the median of all pixel values): [evidence span withheld — anchor status: not_found]
- [other] Derive the lower display bound via Smin = (Ssky - rho_sky * Smax) / (1 - rho_sky) using the fixed ratio rho_sky = 0.001: [evidence span withheld — anchor status: not_found]
- [other] Write Smin and Smax into the SMinSampleValue and SMaxSampleValue TIFF tags of the browser-friendly transcoded image: [evidence span withheld — anchor status: not_found]
- [intro] server-side compression and encoding of the original data to a browser-friendly format remains necessary in order to achieve a satisfactory user experience: "server-side compression and encoding of the original data to a browser-friendly format remains necessary in order to achieve a satisfactory user experience"
- [abstract] It provides access to floating point data at terabyte scales, with the ability to precisely adjust image settings in real-time.: "It provides access to floating point data at terabyte scales, with the ability to precisely adjust image settings in real-time."
- [readme] IIPImage is an advanced high-performance feature-rich image server system for web-based streamed viewing and zooming of ultra high-resolution: [evidence span withheld — anchor status: external_doc]
