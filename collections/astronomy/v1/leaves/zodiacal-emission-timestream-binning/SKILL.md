---
name: zodiacal-emission-timestream-binning
description: Use when you have a simulated zodiacal emission timestream for a scanning
  observer (e.g. DIRBE) and need a binned HEALPix sky map for comparison with calibrated
  individual observations.
when_to_use_negative:
- The goal is a raw timestream comparison against the DIRBE Zodiacal Light Prediction
  Software rather than a sky-map-level comparison with CIO data
- The observer geometry differs from DIRBE and no calibrated individual observations
  exist to bin against
- The wavelength/frequency falls outside the supported 1.25–240 µm infrared and 30–857
  GHz microwave ranges without an acceptable extrapolation
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: ZodiPy
  role: Generates the zodiacal emission timestream for the DIRBE observer via zodipy.Model
    evaluated on SkyCoord pointings
  repo: https://github.com/Cosmoglobe/zodipy
- name: healpy
  role: Bins the timestream into a HEALPix sky map by averaging samples per pixel
- name: Astropy-healpix
  role: Alternative to healpy for HEALPix pixelization when creating the binned map
- name: astropy
  role: Provides SkyCoord objects carrying obstime and galactic-frame coordinates
    and unit handling for the model evaluation
  repo: https://www.astropy.org
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202244133
    title: 'COSMOGLOBE: Simulating zodiacal emission with ZodiPy'
  source_content_hash: sha256:ff2165e0762bef5960e37964695f66202d35385dbc14c58e6e2c67b97f62e83a
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
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# zodiacal-emission-timestream-binning

## Summary

Bin a ZodiPy-predicted zodiacal emission timestream (sampled at the DIRBE observer's pointings) into a HEALPix sky map so it can be compared pixel-by-pixel against the DIRBE calibrated individual observations (CIO).

## When to use

Use when you have a simulated zodiacal emission timestream for a scanning observer (e.g. DIRBE) and need a binned HEALPix sky map for comparison with calibrated individual observations.

## When NOT to use

- The goal is a raw timestream comparison against the DIRBE Zodiacal Light Prediction Software rather than a sky-map-level comparison with CIO data
- The observer geometry differs from DIRBE and no calibrated individual observations exist to bin against
- The wavelength/frequency falls outside the supported 1.25–240 µm infrared and 30–857 GHz microwave ranges without an acceptable extrapolation

## Inputs

- ZodiPy-predicted zodiacal emission timestream sampled at the DIRBE observer's pointings (zodipy.Model evaluated on SkyCoord objects with obstime and galactic frame)
- DIRBE calibrated individual observations (CIO) for the same observing geometry
- HEALPix pixelization (nside) for binning via healpy or Astropy-healpix

## Outputs

- Binned HEALPix sky map of predicted zodiacal emission (written to FITS)
- Per-pixel residual table/statistics comparing the binned ZodiPy map with DIRBE CIO

## How to apply

Evaluate a zodipy.Model (initialized at a wavelength such as 25 micron or over a bandpass) on Astropy SkyCoord objects that carry obstime and galactic-frame coordinates matching the observer's pointings, yielding a predicted zodiacal emission timestream. Bin the timestream into a HEALPix map with healpy or Astropy-healpix by averaging all samples falling in each pixel. Retrieve the matching DIRBE calibrated individual observations (CIO) for the same observing geometry, then compute per-pixel residuals between the binned ZodiPy map and the CIO reference. Write the binned HEALPix map to FITS and summarize residuals in a table; validation rests on the demonstrated agreement between ZodiPy timestreams and those from the COBE DIRBE Zodiacal Light Prediction Software.

## Related tools

- **ZodiPy** (Generates the zodiacal emission timestream for the DIRBE observer via zodipy.Model evaluated on SkyCoord pointings) — https://github.com/Cosmoglobe/zodipy
- **healpy** (Bins the timestream into a HEALPix sky map by averaging samples per pixel)
- **Astropy-healpix** (Alternative to healpy for HEALPix pixelization when creating the binned map)
- **astropy** (Provides SkyCoord objects carrying obstime and galactic-frame coordinates and unit handling for the model evaluation) — https://www.astropy.org

## Examples

```
pip install zodipy; model = zodipy.Model(25 * u.micron); emission = model.evaluate(skycoord)  # skycoord = SkyCoord(lon, lat, obstime=obstimes, frame="galactic")
```

## Evaluation signals

- Per-pixel residuals between the binned ZodiPy map and the DIRBE CIO reference should be small and summarized without systematic structure
- The binned map should be consistent with the known agreement between ZodiPy and DIRBE Zodiacal Light Prediction Software timestreams
- The output map must be a valid HEALPix FITS file with every observed pixel populated by an averaged timestream sample
- Samples entering each pixel must share the same observer geometry/time coverage as the corresponding CIO observations

## Limitations

- ZodiPy covers the DIRBE infrared range 1.25–240 µm and the Planck-extension microwave range 30–857 GHz; other frequencies rely on extrapolation
- Comparison quality depends on the CIO reference and on the sky coordinate/obstime metadata carried by the SkyCoord objects; missing or wrong obstime invalidates the binning
- The validated comparison in the paper is specific to the DIRBE observer and its dust model; other observers inherit the same model assumptions

## Evidence

- [abstract] Binning timestreams into HEALPix maps: "make binned maps of the zodiacal emission as predicted to be observed by DIRBE"
- [abstract] CIO comparison: "compare them with the DIRBE calibrated individual observations (CIO)"
- [methods] Model initialization at wavelength: "# Initialize a zodiacal light model at a wavelength/frequency or over a bandpass
model = zodipy.Model(25 * u.micron)"
- [methods] SkyCoord evaluation: "skycoord = SkyCoord(lon, lat, obstime=obstimes, frame="galactic")"
- [intro] HEALPix tooling: "We can use [healpy](https://healpy.readthedocs.io/en/latest/) or [Astropy-healpix](https://astropy-healpix.readthedocs.io/en/latest/) to create a [`SkyCoord`]"
- [abstract] Timestream agreement validation: "these agree with corresponding timestreams produced with the DIRBE Zodiacal Light Prediction Software"
- [abstract] Frequency coverage: "allows for zodiacal emission predictions at infrared wavelengths in the 1.25–240 µm range and at microwave frequencies in the 30–857 GHz range"
