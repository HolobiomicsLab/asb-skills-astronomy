---
name: dirbe-cio-reference-comparison
description: Use when you have a simulated zodiacal emission timestream for a survey
  observer (e.g. DIRBE) and need a binned HEALPix map that can be validated pixel-by-pixel
  against calibrated time-ordered observations such as the DIRBE CIO dataset.
when_to_use_negative:
- You need raw simulated timestreams only, not a binned map comparable to observational
  data (timestream generation against the DIRBE Zodiacal Light Prediction Software
  is a separate validation step)
- Your reference dataset is not DIRBE CIO or an equivalent calibrated time-ordered
  observation matched to the same observing geometry — per-pixel residuals are meaningless
  without geometric correspondence
- The observer geometry is arbitrary and cannot be matched to CIO pointings; ZodiPy
  supports arbitrary observers but the CIO reference only exists for the DIRBE geometry
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: ZodiPy
  role: Produces the zodiacal emission timestream at the DIRBE observer's pointings
    that is subsequently binned and compared to CIO
  repo: https://github.com/Cosmoglobe/zodipy
- name: healpy
  role: Pixelization of the pointings and binning of the timestream into a HEALPix
    sky map
- name: Astropy-healpix
  role: Alternative to healpy for creating the SkyCoord pointings and HEALPix binning
- name: astropy
  role: SkyCoord objects carrying obstime and galactic-frame coordinates define the
    observer geometry passed to zodipy.Model
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202244133
    title: 'COSMOGLOBE: Simulating zodiacal emission with ZodiPy'
  source_content_hash: sha256:80f8e6da3f616a2d18c98673a7fede2f771313aa0bcdc6ea28440f2625eab614
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Cosmoglobe/zodipy
  tools:
  - Astropy-healpix
  - Python
  - ZodiPy
  - astropy
  - healpy
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# dirbe-cio-reference-comparison

## Summary

Bin a ZodiPy-predicted zodiacal emission timestream sampled at the DIRBE observer's pointings into a HEALPix sky map and compare it against the DIRBE calibrated individual observations (CIO) as an empirical reference.

## When to use

Use when you have a simulated zodiacal emission timestream for a survey observer (e.g. DIRBE) and need a binned HEALPix map that can be validated pixel-by-pixel against calibrated time-ordered observations such as the DIRBE CIO dataset.

## When NOT to use

- You need raw simulated timestreams only, not a binned map comparable to observational data (timestream generation against the DIRBE Zodiacal Light Prediction Software is a separate validation step)
- Your reference dataset is not DIRBE CIO or an equivalent calibrated time-ordered observation matched to the same observing geometry — per-pixel residuals are meaningless without geometric correspondence
- The observer geometry is arbitrary and cannot be matched to CIO pointings; ZodiPy supports arbitrary observers but the CIO reference only exists for the DIRBE geometry

## Inputs

- ZodiPy-predicted zodiacal emission timestream sampled at DIRBE observer pointings (SkyCoord with obstime, galactic frame)
- zodipy.Model initialized at a wavelength/frequency or bandpass (e.g. 25 micron)
- DIRBE calibrated individual observations (CIO) for the same observing geometry
- HEALPix pixelization parameters (nside, via healpy or Astropy-healpix)

## Outputs

- Binned HEALPix sky map of zodiacal emission (FITS file)
- Per-pixel residual map between binned ZodiPy prediction and DIRBE CIO
- Residual summary table/statistic

## How to apply

First evaluate the zodiacal light model on pointings expressed as Astropy SkyCoord objects carrying obstime and galactic-frame coordinates, e.g. model = zodipy.Model(25 * u.micron) initialized at a wavelength, frequency, or bandpass. Bin the resulting timestream into a HEALPix map with healpy or Astropy-healpix, averaging all samples falling in each pixel. Retrieve the matching DIRBE calibrated individual observations (CIO) for the same observing geometry, then compute per-pixel residuals between the binned ZodiPy map and the CIO reference and summarize them as a residual table or statistic. Rationale: the article's authors used exactly this comparison to show that ZodiPy timestreams agree with the DIRBE Zodiacal Light Prediction Software, so per-pixel residuals against CIO are the paper's own validation metric. Write the binned HEALPix map to FITS and the residual summary to a table file so the comparison is reproducible.

## Related tools

- **ZodiPy** (Produces the zodiacal emission timestream at the DIRBE observer's pointings that is subsequently binned and compared to CIO) — https://github.com/Cosmoglobe/zodipy
- **healpy** (Pixelization of the pointings and binning of the timestream into a HEALPix sky map)
- **Astropy-healpix** (Alternative to healpy for creating the SkyCoord pointings and HEALPix binning)
- **astropy** (SkyCoord objects carrying obstime and galactic-frame coordinates define the observer geometry passed to zodipy.Model)

## Examples

```
model = zodipy.Model(25 * u.micron); emission = model.evaluate(SkyCoord(lon, lat, obstime=obstimes, frame='galactic'))
```

## Evaluation signals

- Per-pixel residuals between the binned ZodiPy map and the DIRBE CIO reference should be summarized in a residual table; agreement with DIRBE Zodiacal Light Prediction Software timestreams (as reported in the abstract) is the paper's benchmark
- The binned map must be a valid HEALPix map writable to FITS with consistent nside
- Every timestream sample maps to exactly one HEALPix pixel via the galactic-frame SkyCoord pointings; no pointing should be dropped from binning
- The CIO observations retrieved must match the same obstime/observing geometry as the simulated samples
- The model must be evaluated with a unit-bearing quantity (e.g. 25 * u.micron) or bandpass; bare numbers are invalid

## Limitations

- ZodiPy covers the COBE DIRBE interplanetary dust model and Planck extension over 1.25–240 µm and 30–857 GHz, with extrapolation elsewhere; comparisons outside these ranges rely on extrapolation
- The CIO reference exists only for the DIRBE observer geometry, so this validation workflow does not transfer directly to arbitrary observers without a comparable calibrated dataset
- Agreement was demonstrated against the DIRBE Zodiacal Light Prediction Software for DIRBE-specific geometries; other observers inherit model, not empirical, validation

## Evidence

- [abstract] make binned maps of the zodiacal emission as predicted to be observed by DIRBE and compare them with the DIRBE calibrated individual observations (CIO): "make binned maps of the zodiacal emission as predicted to be observed by DIRBE and compare them with the DIRBE calibrated individual observations (CIO)"
- [abstract] these agree with corresponding timestreams produced with the DIRBE Zodiacal Light Prediction Software: "these agree with corresponding timestreams produced with the DIRBE Zodiacal Light Prediction Software"
- [methods] model = zodipy.Model(25 * u.micron): "# Initialize a zodiacal light model at a wavelength/frequency or over a bandpass
model = zodipy.Model(25 * u.micron)"
- [methods] skycoord = SkyCoord(lon, lat, obstime=obstimes, frame="galactic"): "# Use Astropy's `SkyCoord` object to specify coordinates
lon = [10, 10.1, 10.2] * u.deg ... skycoord = SkyCoord(lon, lat, obstime=obstimes, frame="galactic")"
- [intro] We can use [healpy](https://healpy.readthedocs.io/en/latest/) or [Astropy-healpix](https://astropy-healpix.readthedocs.io/en/latest/) to create a [`SkyCoord`]: "We can use [healpy](https://healpy.readthedocs.io/en/latest/) or [Astropy-healpix](https://astropy-healpix.readthedocs.io/en/latest/) to create a [`SkyCoord`]"
