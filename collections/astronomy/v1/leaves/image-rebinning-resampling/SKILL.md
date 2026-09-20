---
name: image-rebinning-resampling
description: Use when FITS science rasters must be transcoded into a tiled multi-resolution
  TIFF pyramid (e.g. for serving with the IIPImage server in the VisiOmatic web visualization
  stack) via successive 2×2 rebinning.
when_to_use_negative:
- The data already exists as a tiled multi-resolution TIFF or JPEG2000 that the IIPImage
  server can decode directly
- The goal is browser-side rendering from raw FITS (e.g. via a JavaScript FITS library
  like jsFITS) rather than server-side transcoding
- The image is small enough that a single-resolution browser-friendly 8-bit image
  suffices, since the pyramid only pays off for gigapixel-scale scientific imagery
edam_operation: http://edamontology.org/operation_3443
tools:
- name: STIFF
  role: 'Builds the image pyramid: rebins pixels successively 2×2 and stores each
    resolution level in separate TIFF virtual directories in tiled format with constant
    tile size'
- name: visiomatic
  role: Server-side Python environment whose tiled module consumes/produces the tiled
    pyramid; astropy.io.fits is used to load the FITS raster and header before transcoding
  repo: github.com/astromatic/visiomatic
- name: IIPImage (iipsrv)
  role: High-resolution streaming image server that serves the tiled pyramid TIFF
    to web clients; supports 8, 16 and 32 bit per channel including 32-bit floating
    point
  repo: cmarmo/iipsrv-astro
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2014.12.006
    title: Web-based visualization of very large scientific astronomy imagery
  source_content_hash: sha256:4406ea5f0d9d58b382693340b2a964547c36c67173fba21054b0e18e2f055089
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IIPImage (iipsrv)
  - Python
  - STIFF
  - visiomatic
  edam_operation: http://edamontology.org/operation_3443
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# image-rebinning-resampling

## Summary

Builds a multi-resolution image pyramid by successively rebinning a full-resolution raster 2×2 (averaging) and storing each halved level as a separate tiled TIFF virtual directory, with the verbatim FITS header copied into the ImageDescription tag. This is the transcoding step that makes terabyte-scale floating-point astronomy rasters streamable by IIPImage-style servers such as iipsrv.

## When to use

Use when FITS science rasters must be transcoded into a tiled multi-resolution TIFF pyramid (e.g. for serving with the IIPImage server in the VisiOmatic web visualization stack) via successive 2×2 rebinning.

## When NOT to use

- The data already exists as a tiled multi-resolution TIFF or JPEG2000 that the IIPImage server can decode directly
- The goal is browser-side rendering from raw FITS (e.g. via a JavaScript FITS library like jsFITS) rather than server-side transcoding
- The image is small enough that a single-resolution browser-friendly 8-bit image suffices, since the pyramid only pays off for gigapixel-scale scientific imagery

## Inputs

- FITS science image raster (2D pixel array, full bit depth / floating point)
- FITS header string (keywords including WCS information and BUNIT)
- Target tile size for the tiled TIFF pyramid

## Outputs

- Multi-resolution tiled TIFF (e.g. image_pyramid.tif) with one virtual directory per pyramid level
- TIFF ImageDescription tag containing the verbatim FITS header

## How to apply

Load the FITS science raster together with its full header (e.g. with astropy.io.fits), then flip the pixel array along the y-axis to convert from the FITS bottom-left origin to the top-left TIFF convention. Write the full-resolution level as the first TIFF virtual directory using a constant tile size that is kept identical across every level of the pyramid. Successively rebin the array by 2×2 (averaging) and append each halved level as an additional TIFF virtual directory until a minimal top-level directory is reached. Finally, copy the verbatim FITS header string into the TIFF ImageDescription tag so WCS information and keywords such as BUNIT remain available to the client; the resulting image_pyramid.tif is what the IIPImage server streams to the VisiOmatic client.

## Related tools

- **STIFF** (Builds the image pyramid: rebins pixels successively 2×2 and stores each resolution level in separate TIFF virtual directories in tiled format with constant tile size)
- **visiomatic** (Server-side Python environment whose tiled module consumes/produces the tiled pyramid; astropy.io.fits is used to load the FITS raster and header before transcoding) — github.com/astromatic/visiomatic
- **IIPImage (iipsrv)** (High-resolution streaming image server that serves the tiled pyramid TIFF to web clients; supports 8, 16 and 32 bit per channel including 32-bit floating point) — cmarmo/iipsrv-astro

## Examples

```
from astropy.io import fits; data = fits.getdata('science.fits')[::-1, :]  # y-flip FITS bottom-left origin to TIFF top-left before writing pyramid levels
```

## Evaluation signals

- Reopen image_pyramid.tif with a TIFF reader and verify the number of directories equals ceil(log2(image_width / top_level_width)) + 1
- Verify that image dimensions shrink by exactly a factor of 2 from one TIFF directory to the next
- Verify that the tile size is identical across all pyramid directories
- Verify pixel orientation matches the y-flipped FITS raster (bottom-left FITS origin converted to top-left TIFF convention)
- Verify the ImageDescription tag matches the original FITS header verbatim, preserving keywords such as BUNIT

## Limitations

- The 2×2 rebinning loses spatial detail at deeper pyramid levels; the top-level directory is reached only after successive halving, so the level count is bounded by the image width
- Browser engines display gamma-encoded 8-bit-per-channel images, so the transcoding and server-side encoding to a browser-friendly format remain necessary regardless of the pyramid construction
- The FITS-to-TIFF origin convention differs (bottom-left vs top-left); skipping the y-axis flip corrupts pixel orientation
- iipsrv depends on libtiff, zlib and IJG JPEG libraries, and JPEG2000 decoding requires the non-open-source Kakadu SDK or the experimental OpenJPEG path

## Evidence

- [methods] STIFF builds the multi-resolution structure as an image 'pyramid' in which pixels are successively rebinned 2×2 and stored in separate TIFF virtual directories in tiled format, with constant tile size across all resolution levels.: [evidence span withheld — anchor status: not_found]
