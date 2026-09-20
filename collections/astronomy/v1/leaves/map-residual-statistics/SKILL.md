---
name: map-residual-statistics
description: Use when you have a predicted-emission timestream sampled at an instrument's
  pointings (e.g. ZodiPy output for the DIRBE observer) and need a binned HEALPix
  map comparable to reference observations such as DIRBE CIO, plus per-pixel residual
  statistics.
when_to_use_negative:
- The goal is a raw timestream-vs-timestream comparison against the DIRBE Zodiacal
  Light Prediction Software rather than a binned map-level residual analysis
- The observation sampling is not on an instrument pointing sequence that can be pixelized
  into HEALPix (no positional metadata to bin by)
- The emission prediction is outside ZodiPy's supported ranges (1.25-240 um infrared
  or 30-857 GHz microwave)
tools:
- name: ZodiPy
  role: Produces the zodiacal emission timestream predicted for the DIRBE observer
    that is subsequently binned into a HEALPix map
  repo: github.com/Cosmoglobe/zodipy
- name: healpy
  role: Pixelization/binning of the timestream into a HEALPix sky map and FITS output
- name: Astropy-healpix
  role: Alternative to healpy for creating the SkyCoord/pixelization used in binning
- name: astropy
  role: Carries observing geometry (SkyCoord with obstime, galactic frame) and units
    for the model evaluation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202244133
    title: 'COSMOGLOBE: Simulating zodiacal emission with ZodiPy'
  source_content_hash: sha256:939767d5b234f0f41c7720e2f4f0464905e446d89c570a381c06cc6805806801
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Astropy-healpix
  - Python
  - ZodiPy
  - astropy
  - healpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# map-residual-statistics

## Summary

Bins an observer-sampled zodiacal-emission timestream into a HEALPix sky map (using healpy/Astropy-healpix) and computes per-pixel residuals against a reference dataset such as the DIRBE calibrated individual observations (CIO), summarizing agreement as a residual table/statistic.

## When to use

Use when you have a predicted-emission timestream sampled at an instrument's pointings (e.g. ZodiPy output for the DIRBE observer) and need a binned HEALPix map comparable to reference observations such as DIRBE CIO, plus per-pixel residual statistics.

## When NOT to use

- The goal is a raw timestream-vs-timestream comparison against the DIRBE Zodiacal Light Prediction Software rather than a binned map-level residual analysis
- The observation sampling is not on an instrument pointing sequence that can be pixelized into HEALPix (no positional metadata to bin by)
- The emission prediction is outside ZodiPy's supported ranges (1.25-240 um infrared or 30-857 GHz microwave)

## Inputs

- ZodiPy-predicted zodiacal emission timestream sampled at the DIRBE observer's pointings (SkyCoord objects with obstime, galactic frame)
- DIRBE calibrated individual observations (CIO) for the same observing geometry
- zodipy.Model initialized at a wavelength/frequency or bandpass (e.g. 25 micron)

## Outputs

- Binned HEALPix sky map of zodiacal emission written to a FITS file
- Per-pixel residual table/statistic comparing the binned ZodiPy map against DIRBE CIO, written to a table file

## How to apply

Load the ZodiPy-predicted zodiacal emission timestream sampled at the DIRBE observer's pointings, where coordinates are carried as Astropy SkyCoord objects with obstime in the galactic frame. Bin the timestream into a HEALPix sky map using healpy or Astropy-healpix, averaging the samples that fall into each pixel. Retrieve the matching DIRBE calibrated individual observations (CIO) for the same observing geometry and compute per-pixel residuals between the binned ZodiPy map and the CIO reference. Summarize the residuals as a residual table/statistic, and write the binned HEALPix map to a FITS file and the residual summary to a table file.

## Related tools

- **ZodiPy** (Produces the zodiacal emission timestream predicted for the DIRBE observer that is subsequently binned into a HEALPix map) — github.com/Cosmoglobe/zodipy
- **healpy** (Pixelization/binning of the timestream into a HEALPix sky map and FITS output)
- **Astropy-healpix** (Alternative to healpy for creating the SkyCoord/pixelization used in binning)
- **astropy** (Carries observing geometry (SkyCoord with obstime, galactic frame) and units for the model evaluation)

## Examples

```
from astropy.coordinates import SkyCoord; skycoord = SkyCoord(lon, lat, obstime=obstimes, frame="galactic"); model = zodipy.Model(25 * u.micron)
```

## Evaluation signals

- The binned HEALPix map's underlying timestream agrees with timestreams produced by the DIRBE Zodiacal Light Prediction Software
- Per-pixel residuals between the binned ZodiPy map and DIRBE CIO are tabulated and summarized with finite statistics
- The output HEALPix map is written as a valid FITS file and the residual summary as a valid table file
- Every timestream sample is assigned to exactly one HEALPix pixel and each pixel value is the average of the samples falling in it

## Limitations

- ZodiPy covers infrared wavelengths of 1.25-240 um and microwave frequencies of 30-857 GHz, with extrapolation needed for other frequencies/wavelengths
- Requires Astropy >= 5.0.1 as a dependency
- Binning averages samples that fall in each pixel, so fine structure below the HEALPix pixel scale is not preserved in the map-level residual comparison

## Evidence

- [abstract] make binned maps of the zodiacal emission as predicted to be observed by DIRBE: "make binned maps of the zodiacal emission as predicted to be observed by DIRBE"
- [abstract] compare them with the DIRBE calibrated individual observations (CIO): "compare them with the DIRBE calibrated individual observations (CIO)"
- [methods] binning into HEALPix map: "# Use Astropy's `SkyCoord` object to specify coordinates"
- [methods] initialize model at wavelength or bandpass: "# Initialize a zodiacal light model at a wavelength/frequency or over a bandpass
model = zodipy.Model(25 * u.micron)"
- [abstract] timestream agreement with DIRBE software: "these agree with corresponding timestreams produced with the DIRBE Zodiacal Light Prediction Software"
