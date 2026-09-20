---
name: zodiacal-light-emission-evaluation
description: Use when you need to compute or verify zodiacal emission intensities
  (MJy/sr) for specific sky coordinates, observation times, and a solar system observer
  position, in the DIRBE infrared 1.25-240 µm or microwave 30-857 GHz range.
when_to_use_negative:
- The wavelength/bandpass lies outside ZodiPy's supported ranges (1.25-240 µm infrared
  or 30-857 GHz microwave) and no extrapolation is acceptable.
- The goal is full-sky map-making or timestream simulation for an instrument like
  DIRBE rather than point line-of-sight evaluations.
- The observer is outside the solar system or no ephemeris is available to resolve
  an obspos body name or cartesian position.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: ZodiPy
  role: Provides the zodiacal light model (COBE DIRBE interplanetary dust model with
    Planck extension) and the Model.evaluate line-of-sight computation with obspos
    support
  repo: https://github.com/Cosmoglobe/zodipy
- name: astropy
  role: Supplies SkyCoord, Time, and units used to specify coordinates, obstime, and
    the 25 micron wavelength
- name: healpy
  role: Can be used to create the SkyCoord grid of directions over which the model
    is evaluated
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202244133
    title: 'COSMOGLOBE: Simulating zodiacal emission with ZodiPy'
  source_content_hash: sha256:02c5539188ddf0f657f7f9aeed5c9006cb638e73e6477b425f6d6ebe3591d8bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Cosmoglobe/zodipy
  tools:
  - Python
  - ZodiPy
  - astropy
  - healpy
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# zodiacal-light-emission-evaluation

## Summary

Point-evaluation of zodiacal light emission with ZodiPy: initialize a DIRBE-based dust model at a fixed wavelength (e.g. 25 micron), build Astropy SkyCoord line-of-sight grids with obstime, and reproduce documented MJy/sr intensities for default Earth-center and non-Earth observers (e.g. Mars).

## When to use

Use when you need to compute or verify zodiacal emission intensities (MJy/sr) for specific sky coordinates, observation times, and a solar system observer position, in the DIRBE infrared 1.25-240 µm or microwave 30-857 GHz range.

## When NOT to use

- The wavelength/bandpass lies outside ZodiPy's supported ranges (1.25-240 µm infrared or 30-857 GHz microwave) and no extrapolation is acceptable.
- The goal is full-sky map-making or timestream simulation for an instrument like DIRBE rather than point line-of-sight evaluations.
- The observer is outside the solar system or no ephemeris is available to resolve an obspos body name or cartesian position.

## Inputs

- Fixed wavelength or bandpass within DIRBE coverage (e.g. 25 * u.micron)
- Astropy SkyCoord objects with galactic lon/lat and obstime (astropy Time)
- Observer position spec: solar system body name (e.g. 'mars') or heliocentric ecliptic cartesian position via obspos
- Solar system ephemeris data for non-Earth observer positions

## Outputs

- Zodiacal emission intensity values in MJy/sr per evaluated sky coordinate/time
- point_evaluations.csv table of coordinates, obstime, observer position, computed value, and reference value

## How to apply

Install ZodiPy via pip (Python >= 3.9, Astropy >= 5.0.1 as dependency) and initialize a model at a fixed wavelength in the DIRBE-covered range, e.g. zodipy.Model(25 * u.micron). Build Astropy SkyCoord objects from galactic longitudes/latitudes with obstime set via astropy Time (e.g. 2020-01-01 or 2022-01-01 timestamps), then call model.evaluate(skycoord) to get line-of-sight intensities. To move the observer off Earth, pass the obspos keyword with either a solar system body name (e.g. obspos='mars', resolved through the solar system ephemeris) or a heliocentric ecliptic cartesian position. Compare each computed intensity against documented reference values — 27.52410841/27.66572294/27.81251906 MJy/sr for the galactic 40/60 deg sequence at Earth, 25.08189292 MJy/sr for the default Earth-center observer at 25 micron, and 8.36985535 MJy/sr for obspos='mars' at 25 micron — within a small numerical tolerance, and record the comparison table (coordinates, obstime, observer position, computed vs reference value) to point_evaluations.csv. Correctness is judged by agreement of computed values with these references and, at scale, by agreement of ZodiPy DIRBE timestreams with those from the DIRBE Zodiacal Light Prediction Software.

## Related tools

- **ZodiPy** (Provides the zodiacal light model (COBE DIRBE interplanetary dust model with Planck extension) and the Model.evaluate line-of-sight computation with obspos support) — https://github.com/Cosmoglobe/zodipy
- **astropy** (Supplies SkyCoord, Time, and units used to specify coordinates, obstime, and the 25 micron wavelength)
- **healpy** (Can be used to create the SkyCoord grid of directions over which the model is evaluated)

## Examples

```
model = zodipy.Model(25 * u.micron); skycoord = SkyCoord(lon, lat, obstime=obstimes, frame="galactic"); emission = model.evaluate(skycoord, obspos='mars')
```

## Evaluation signals

- model.evaluate at galactic 40/60 deg, obstime 2020-01-01, returns the 27.52410841/27.66572294/27.81251906 MJy/sr sequence within tolerance
- Default Earth-center observer at 25 micron yields 25.08189292 MJy/sr
- obspos='mars' evaluation at 25 micron yields 8.36985535 MJy/sr
- point_evaluations.csv contains one row per evaluation with matching computed and reference values within small numerical tolerance
- For full timestreams, ZodiPy output agrees with DIRBE Zodiacal Light Prediction Software timestreams

## Limitations

- ZodiPy covers only 1.25-240 µm (DIRBE model) and 30-857 GHz (Planck extension), with extrapolation needed outside these ranges.
- Non-Earth observer positions depend on the solar system ephemeris; obspos accepts only body names or heliocentric ecliptic cartesian positions.
- Reference values are tied to specific obstimes and coordinates; evaluations at other times/points will not match the documented numbers.

## Evidence

- [methods] evaluate accepts obspos keyword giving body name or heliocentric ecliptic cartesian position: [evidence span withheld — anchor status: not_found]
- [methods] initialize model at fixed wavelength: "model = zodipy.Model(25 * u.micron)"
- [methods] SkyCoord with obstime and galactic frame then evaluate: "skycoord = SkyCoord(lon, lat, obstime=obstimes, frame="galactic")"
- [abstract] supported wavelength/frequency coverage: "allows for zodiacal emission predictions at infrared wavelengths in the 1.25–240 µm range and at microwave frequencies in the 30–857 GHz range"
- [abstract] validation against DIRBE software: "these agree with corresponding timestreams produced with the DIRBE Zodiacal Light Prediction Software"
