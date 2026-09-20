---
name: zodiacal-emission-extrapolation-beyond-model-validity-range
description: Use when you need zodiacal emission forecasts for a future experiment
  whose observing bands fall below 1.25 µm or above 240 µm (or below 30 GHz / above
  857 GHz), outside the DIRBE/Planck model ranges built into ZodiPy.
when_to_use_negative:
- The target observing bands already lie within the built-in 1.25–240 µm / 30–857
  GHz validity ranges, where interpolation alone suffices and extrapolation is unnecessary.
- The analysis requires zodiacal emission timestreams validated against an existing
  instrument such as DIRBE rather than out-of-range forecasts.
- You need a physically recalibrated dust model, not a linear extrapolation of built-in
  model parameters, since extrapolation is a forecast tool, not a validated model
  extension.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3353
tools:
- name: ZodiPy
  role: Provides the zodiacal light model whose `extrapolate` keyword enables linear
    extrapolation of frequency/wavelength-dependent parameters; model.evaluate(skycoord)
    yields intensities in MJy/sr
  repo: github.com/Cosmoglobe/zodipy
- name: Astropy
  role: Supplies astropy.units for specifying out-of-range wavelengths/frequencies
    and astropy.coordinates.SkyCoord (galactic frame, with obstimes) for observer
    positions
  repo: https://www.astropy.org
- name: healpy
  role: Optional alternative (with Astropy-healpix) for creating the SkyCoord pixelization
    used to evaluate the model
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202244133
    title: 'COSMOGLOBE: Simulating zodiacal emission with ZodiPy'
  source_content_hash: sha256:e88f0df418a18d12946b8c4369e24713ce291ae3630f0cd8228f0dfcd86866ca
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Astropy
  - Python
  - ZodiPy
  - healpy
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3353
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# zodiacal-emission-extrapolation-beyond-model-validity-range

## Summary

A ZodiPy-based technique for forecasting zodiacal emission at observing frequencies/wavelengths outside the built-in model validity ranges (1.25–240 µm and 30–857 GHz) by enabling linear extrapolation of the frequency/wavelength-dependent model parameters.

## When to use

Use when you need zodiacal emission forecasts for a future experiment whose observing bands fall below 1.25 µm or above 240 µm (or below 30 GHz / above 857 GHz), outside the DIRBE/Planck model ranges built into ZodiPy.

## When NOT to use

- The target observing bands already lie within the built-in 1.25–240 µm / 30–857 GHz validity ranges, where interpolation alone suffices and extrapolation is unnecessary.
- The analysis requires zodiacal emission timestreams validated against an existing instrument such as DIRBE rather than out-of-range forecasts.
- You need a physically recalibrated dust model, not a linear extrapolation of built-in model parameters, since extrapolation is a forecast tool, not a validated model extension.

## Inputs

- Target observing frequencies/wavelengths outside 1.25–240 µm and 30–857 GHz, expressed as astropy.units Quantities
- Solar System observer positions and obstimes as astropy.coordinates.SkyCoord in the galactic frame
- ZodiPy built-in zodiacal light models (COBE DIRBE interplanetary dust model and/or Planck extension)

## Outputs

- Predicted zodiacal emission intensities in MJy/sr at out-of-range frequencies
- Forecast table zodipy_extrapolation_forecast.csv with frequency and intensity (MJy/sr) columns

## How to apply

Initialize a zodipy.Model at each out-of-range target frequency or wavelength using astropy.units, mirroring the documented pattern `model = zodipy.Model(25 * u.micron)`, and set the Model's `extrapolate` keyword to True so that the frequency/wavelength-dependent model parameters are linearly extrapolated beyond their tabulated bounds. Specify Solar System observer positions and obstimes with astropy.coordinates.SkyCoord in the galactic frame, following the article's lon/lat/obstime setup. Evaluate the model via model.evaluate(skycoord) and collect the returned intensities, which are given in MJy/sr. Assemble the frequency/wavelength and intensity pairs into a forecast table (zodipy_extrapolation_forecast.csv), then validate that all tabulated frequencies lie strictly outside the 1.25–240 µm / 30–857 GHz validity ranges and that the intensity column is present in MJy/sr. The rationale is that the DIRBE and Planck dust models only parameterize emissivity at their native bands, so linear extrapolation of those parameters is the mechanism ZodiPy provides to support zodiacal light forecasting for future experiments.

## Related tools

- **ZodiPy** (Provides the zodiacal light model whose `extrapolate` keyword enables linear extrapolation of frequency/wavelength-dependent parameters; model.evaluate(skycoord) yields intensities in MJy/sr) — github.com/Cosmoglobe/zodipy
- **Astropy** (Supplies astropy.units for specifying out-of-range wavelengths/frequencies and astropy.coordinates.SkyCoord (galactic frame, with obstimes) for observer positions) — https://www.astropy.org
- **healpy** (Optional alternative (with Astropy-healpix) for creating the SkyCoord pixelization used to evaluate the model)

## Examples

```
model = zodipy.Model(300 * u.micron, extrapolate=True); intensity = model.evaluate(SkyCoord(lon, lat, obstime=obstimes, frame='galactic'))
```

## Evaluation signals

- The output CSV zodipy_extrapolation_forecast.csv exists and contains a frequency column and an intensity column expressed in MJy/sr.
- Every tabulated frequency lies strictly outside the 1.25–240 µm (and 30–857 GHz) built-in model validity ranges.
- Model evaluation succeeds without ZodiPy raising an out-of-range error, confirming the `extrapolate=True` keyword was accepted.
- Intensities are physically plausible (positive, finite MJy/sr values) and vary smoothly across the extrapolated frequency grid.

## Limitations

- Extrapolation is linear interpolation/extrapolation of model parameters, so forecasts far outside the tabulated range carry unquantified uncertainty and are not validated against measurements.
- The built-in ZodiPy models only natively cover 1.25–240 µm and 30–857 GHz; results outside these bands depend entirely on the extrapolation behavior.
- Validation against observations is established only for in-range comparisons (ZodiPy DIRBE timestreams agree with the DIRBE Zodiacal Light Prediction Software); no such check exists for extrapolated bands.

## Evidence

- [abstract] Built-in model ranges and extrapolation capability: "allows for zodiacal emission predictions at infrared wavelengths in the 1.25–240 µm range and at microwave frequencies in the 30–857 GHz range"
- [intro] Extrapolation purpose for future experiments: "assisting in astrophysical data analysis and zodiacal light forecasting for future experiments."
- [methods] Model initialization pattern: "# Initialize a zodiacal light model at a wavelength/frequency or over a bandpass
model = zodipy.Model(25 * u.micron)"
- [methods] SkyCoord evaluation pattern: "# Use Astropy's `SkyCoord` object to specify coordinates"
