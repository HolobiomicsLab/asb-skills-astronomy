---
name: display-scaling-parameter-derivation
description: Use when a FITS science image must be transcoded to a browser-friendly
  format (e.g., TIFF tiles for IIPImage) and global display cut-offs Smin/Smax must
  be written to TIFF sample-value tags.
when_to_use_negative:
- The input is already an 8-bit display-ready image where static storage limits interaction
  to passive visualization and no re-scaling metadata is needed
- Per-tile or real-time dynamic scaling is required instead of global static cut-offs
  baked into TIFF tags
- The median of all pixel values is not a valid sky-background estimate (e.g., strongly
  crowded or extended-source fields), violating the Smin formula's assumption
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: JavaScript
  role: Client-side engine of the W3C standards-based web client that consumes the
    transcoded 8-bit tiles whose display scale was fixed by the derived Smin/Smax
    tags
  repo: TPZF/RTWeb3D
- name: iipsrv-astro
  role: IIPImage server that streams the transcoded TIFF imagery and relies on the
    SMinSampleValue/SMaxSampleValue tags for display scaling of scientific imagery
  repo: cmarmo/iipsrv-astro
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2014.12.006
    title: Web-based visualization of very large scientific astronomy imagery
  source_content_hash: sha256:366cabb14480c636059e99dc14a93821367a40b6da61188c43534239594e4119
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TPZF/RTWeb3D
  tools:
  - JavaScript
  - iipsrv-astro
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# display-scaling-parameter-derivation

## Summary

Derives global display scaling metadata (Smin, Smax) for transcoding FITS astronomy imagery into a browser-friendly format consumed by the IIPImage server, following the STIFF transcoding convention: Smax at the 999th permil of the pixel histogram and Smin placing the sky background at rho_sky = 0.001 of maximum display radiant emittance.

## When to use

Use when a FITS science image must be transcoded to a browser-friendly format (e.g., TIFF tiles for IIPImage) and global display cut-offs Smin/Smax must be written to TIFF sample-value tags.

## When NOT to use

- The input is already an 8-bit display-ready image where static storage limits interaction to passive visualization and no re-scaling metadata is needed
- Per-tile or real-time dynamic scaling is required instead of global static cut-offs baked into TIFF tags
- The median of all pixel values is not a valid sky-background estimate (e.g., strongly crowded or extended-source fields), violating the Smin formula's assumption

## Inputs

- FITS science image pixel array (full 16-bit or 32-bit floating point data consumed by the transcoding step)

## Outputs

- Smin and Smax values written to the SMinSampleValue and SMaxSampleValue TIFF tags of the browser-friendly transcoded image
- Exported triplet (Smin, Ssky, Smax) in scale_parameters

## How to apply

Load the FITS science image pixel array (the same input consumed by the transcoding step) and accumulate the full pixel-value histogram. Set the upper display bound Smax to the 999th permil (99.9th percentile) of that histogram, per the STIFF default. Estimate the sky level Ssky as the median of all pixel values, then derive the lower bound via Smin = (Ssky - rho_sky * Smax) / (1 - rho_sky) with the fixed ratio rho_sky = 0.001, so the sky background appears at about 0.001 of the maximum display radiant emittance. Write Smin and Smax into the SMinSampleValue and SMaxSampleValue TIFF tags of the transcoded image so the IIPImage server renders with consistent contrast, and export the triplet (Smin, Ssky, Smax) to scale_parameters. The rationale is that browser engines display gamma-encoded 8-bit imagery, so server-side transcoding must bake the scaling into the file metadata.

## Related tools

- **JavaScript** (Client-side engine of the W3C standards-based web client that consumes the transcoded 8-bit tiles whose display scale was fixed by the derived Smin/Smax tags) — TPZF/RTWeb3D
- **iipsrv-astro** (IIPImage server that streams the transcoded TIFF imagery and relies on the SMinSampleValue/SMaxSampleValue tags for display scaling of scientific imagery) — cmarmo/iipsrv-astro

## Examples

```
from astropy.io import fits; import numpy as np; d = fits.getdata('img.fits').ravel(); smax = np.percentile(d, 99.9); ssky = np.median(d); smin = (ssky - 0.001*smax)/(1-0.001)
```

## Evaluation signals

- Inequality invariant Smin < Ssky < Smax holds for the exported triplet
- SMinSampleValue and SMaxSampleValue TIFF tags contain exactly the computed Smin and Smax
- Smax equals the 99.9th percentile (999th permil) of the accumulated FITS pixel-value histogram
- Smin reproduces (Ssky - 0.001 * Smax) / (1 - 0.001) to numerical precision with rho_sky = 0.001
- Ssky equals the median of all pixel values of the input FITS array

## Limitations

- The rho_sky = 0.001 ratio and the sky = median assumption are STIFF defaults; they may mis-scale images where the median is not representative of the sky background
- Smax fixed at the 999th permil clips the brightest 0.1% of pixels, which may truncate saturated or bright-object display
- The derived parameters are static global metadata; they do not provide the real-time image setting adjustment the server-side streaming system offers at full 32-bit precision

## Evidence

- [other] STIFF sets Smax to the 999th permil of the image histogram by default: [evidence span withheld — anchor status: not_found]
- [other] Derive the lower display bound via Smin = (Ssky - rho_sky * Smax) / (1 - rho_sky) using the fixed ratio rho_sky = 0.001: [evidence span withheld — anchor status: not_found]
- [other] accept the output when Smin < Ssky < Smax holds: [evidence span withheld — anchor status: not_found]
- [intro] server-side compression and encoding of the original data to a browser-friendly format remains necessary: "server-side compression and encoding of the original data to a browser-friendly format remains necessary in order to achieve a satisfactory user experience"
