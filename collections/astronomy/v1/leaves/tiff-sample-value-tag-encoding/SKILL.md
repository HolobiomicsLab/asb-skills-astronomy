---
name: tiff-sample-value-tag-encoding
description: Use when transcoding full-precision FITS science imagery to a browser-friendly
  TIFF for the IIPImage server and the display bounds Smin/Smax must be computed from
  the pixel histogram and stored in SMinSampleValue/SMaxSampleValue tags.
when_to_use_negative:
- Pixels are converted entirely in-browser from FITS via JavaScript (e.g., jsFITS),
  so no server-side TIFF sample-value tags are produced.
- The target is a static 8-bit compressed image for passive visualization with a pre-fixed
  display range rather than server-side transcoding of full-precision data.
- The frame lacks a dominant sky background, making the median-based Ssky estimate
  a meaningless anchor for the lower display bound.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: STIFF
  role: Reference transcoding tool whose default Smin/Smax display-scale heuristic
    this skill reconstructs
- name: iipsrv (IIPImage server)
  role: Streams the browser-friendly TIFF tiles to web clients, consuming the SMinSampleValue/SMaxSampleValue
    display bounds
  repo: cmarmo/iipsrv-astro
- name: astropy.io.fits
  role: Loads the FITS science pixel array used to accumulate the histogram for Smax
    and the median for Ssky
- name: jsFITS
  role: JavaScript FITS reader representing the in-browser alternative path where
    no TIFF tags are written
  repo: slowe/jsFITS
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2014.12.006
    title: Web-based visualization of very large scientific astronomy imagery
  source_content_hash: sha256:4a5e767e7db1ba12940d1b618e8d6b2fdc204f8b65f981fcc12927f127e5c7cc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/cmarmo/iipsrv-astro
  tools:
  - STIFF
  - astropy.io.fits
  - iipsrv (IIPImage server)
  - jsFITS
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# tiff-sample-value-tag-encoding

## Summary

Reconstructs the STIFF transcoding tool's default display-scale heuristic: Smax set to the 999th permil of the FITS pixel-value histogram and Smin derived from the sky-median Ssky at rho_sky = 0.001, then encodes both into the SMinSampleValue/SMaxSampleValue TIFF tags of the browser-friendly image streamed by the IIPImage server.

## When to use

Use when transcoding full-precision FITS science imagery to a browser-friendly TIFF for the IIPImage server and the display bounds Smin/Smax must be computed from the pixel histogram and stored in SMinSampleValue/SMaxSampleValue tags.

## When NOT to use

- Pixels are converted entirely in-browser from FITS via JavaScript (e.g., jsFITS), so no server-side TIFF sample-value tags are produced.
- The target is a static 8-bit compressed image for passive visualization with a pre-fixed display range rather than server-side transcoding of full-precision data.
- The frame lacks a dominant sky background, making the median-based Ssky estimate a meaningless anchor for the lower display bound.

## Inputs

- FITS science image pixel array (2-D, loaded e.g. with astropy.io.fits)
- Fixed display ratio rho_sky = 0.001 (sky at 0.001 of maximum display radiant emittance)

## Outputs

- SMinSampleValue and SMaxSampleValue TIFF tags written to the browser-friendly transcoded image
- Exported triplet (Smin, Ssky, Smax) in scale_parameters

## How to apply

Load the FITS science pixel array (e.g., via astropy.io.fits) and accumulate the full pixel-value histogram, then set the upper display bound Smax to the 999th permil (99.9th percentile) of that histogram. Estimate the sky background Ssky as the median of all pixel values and derive the lower bound via Smin = (Ssky - rho_sky * Smax) / (1 - rho_sky) with the fixed ratio rho_sky = 0.001, so the sky sits at 0.001 of the maximum display radiant emittance. Write the computed Smin and Smax into the SMinSampleValue and SMaxSampleValue TIFF tags of the browser-friendly transcoded image, because browsers natively render only gamma-encoded 8-bit images, making server-side encoding of the display mapping necessary. Export the triplet (Smin, Ssky, Smax) to scale_parameters for downstream reuse. Accept the result only when Smin < Ssky < Smax holds and read-back of the two sample-value tags reproduces the computed Smin and Smax exactly.

## Related tools

- **STIFF** (Reference transcoding tool whose default Smin/Smax display-scale heuristic this skill reconstructs)
- **iipsrv (IIPImage server)** (Streams the browser-friendly TIFF tiles to web clients, consuming the SMinSampleValue/SMaxSampleValue display bounds) — cmarmo/iipsrv-astro
- **astropy.io.fits** (Loads the FITS science pixel array used to accumulate the histogram for Smax and the median for Ssky)
- **jsFITS** (JavaScript FITS reader representing the in-browser alternative path where no TIFF tags are written) — slowe/jsFITS

## Examples

```
from astropy.io import fits; import numpy as np; px = fits.getdata('img.fits'); Smax = np.percentile(px, 99.9); Ssky = np.median(px); Smin = (Ssky - 0.001*Smax) / 0.999
```

## Evaluation signals

- Ordering invariant Smin < Ssky < Smax holds for the exported triplet.
- Read-back of SMinSampleValue and SMaxSampleValue from the transcoded TIFF reproduces the computed Smin and Smax exactly.
- Smax equals the 999th permil (99.9th percentile) of the accumulated FITS pixel-value histogram.
- Linear display mapping places the sky level at target: (Ssky - Smin) / (Smax - Smin) = rho_sky = 0.001.
- scale_parameters contains all three of Smin, Ssky, Smax, with Ssky equal to the median of all pixel values.

## Limitations

- Setting Smax at the 999th permil intentionally truncates the brightest 0.1% of pixels, so peak/bright-source cores are saturated by design.
- Ssky is taken as the median of all pixel values, which presumes a dominant sky background typical of astronomical frames; other imagery yields a poorly placed lower bound.
- The tags only take effect in consumers that honour them (e.g., the IIPImage server, which requires libtiff, zlib and IJG JPEG); flattening to a plain 8-bit export discards the display mapping.

## Evidence

- [other] STIFF default Smax percentile and Ssky definition: [evidence span withheld — anchor status: not_found]
- [other] Smin derivation formula with fixed ratio: [evidence span withheld — anchor status: not_found]
- [readme] IIPImage server consumes the transcoded tiles: [evidence span withheld — anchor status: external_doc]
