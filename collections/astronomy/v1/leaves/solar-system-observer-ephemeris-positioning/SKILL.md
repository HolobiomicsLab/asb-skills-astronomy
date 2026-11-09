---
name: solar-system-observer-ephemeris-positioning
description: Use when simulating zodiacal emission as seen by a non-Earth observer
  (e.g. Mars), or when validating Model.evaluate point values against documented reference
  intensities in MJy/sr.
when_to_use_negative:
- The analysis targets microwave frequencies outside the 30–857 GHz band or infrared
  wavelengths outside the 1.25–240 µm DIRBE model range
- The goal is a full binned sky map or DIRBE timestream simulation rather than point
  evaluations of Model.evaluate
- The observer should remain at Earth's center (the ZodiPy default) and no body-name
  or cartesian obspos override is needed
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3412
tools:
- name: ZodiPy
  role: Provides Model(25 * u.micron) and the evaluate(skycoord, obspos=...) mechanism
    that resolves the observer position and computes line-of-sight zodiacal emission
  repo: https://github.com/Cosmoglobe/zodipy
- name: astropy
  role: Supplies SkyCoord, Time, and units used to define galactic coordinates, obstimes,
    and the 25 micron wavelength; ZodiPy requires Astropy >= 5.0.1
- name: healpy
  role: Optional helper to create a SkyCoord grid of pointing directions for the observer
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202244133
    title: 'COSMOGLOBE: Simulating zodiacal emission with ZodiPy'
  source_content_hash: sha256:22becac910921da14cc7421b33c74d6c9213dac2e26e9ea5e4cb4022ebb23427
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Cosmoglobe/zodipy
  tools:
  - Python
  - ZodiPy
  - astropy
  - healpy
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_3412
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# solar-system-observer-ephemeris-positioning

## Summary

Places the ZodiPy zodiacal-emission observer at an arbitrary Solar System location by passing the obspos keyword (a solar system body name like 'mars' or a heliocentric ecliptic cartesian position) to Model.evaluate, so line-of-sight emission is computed from that observer's ephemeris-resolved position instead of the default Earth center.

## When to use

Use when simulating zodiacal emission as seen by a non-Earth observer (e.g. Mars), or when validating Model.evaluate point values against documented reference intensities in MJy/sr.

## When NOT to use

- The analysis targets microwave frequencies outside the 30–857 GHz band or infrared wavelengths outside the 1.25–240 µm DIRBE model range
- The goal is a full binned sky map or DIRBE timestream simulation rather than point evaluations of Model.evaluate
- The observer should remain at Earth's center (the ZodiPy default) and no body-name or cartesian obspos override is needed

## Inputs

- Astropy SkyCoord with galactic longitudes/latitudes and obstime=Time(...) timestamps
- obspos keyword: solar system body name (e.g. 'mars') or heliocentric ecliptic cartesian position
- Wavelength or bandpass within the DIRBE range, e.g. 25 * u.micron

## Outputs

- Zodiacal emission intensities in MJy/sr per SkyCoord point and observer position
- point_evaluations.csv with coordinates, obstime, observer position, computed value, and reference value

## How to apply

Install ZodiPy via pip (Python >= 3.9, Astropy >= 5.0.1) and initialize the model at a fixed wavelength inside the DIRBE-covered infrared range, e.g. zodipy.Model(25 * u.micron). Build Astropy SkyCoord objects with galactic longitude/latitude and obstime (e.g. Time('2020-01-01') or 2022-01-01 12:00-12:02 timestamps). To relocate the observer, pass obspos to model.evaluate — either a solar system body name resolved via the solar system ephemeris (obspos='mars') or an explicit heliocentric ecliptic cartesian position. The default obspos is the Earth center; compare the Mars-observer intensity against the documented 8.36985535 MJy/sr at 25 micron and the default against 25.08189292 MJy/sr within a small numerical tolerance. Record coordinates, obstime, observer position, computed value, and reference value per point (e.g. to point_evaluations.csv).

## Related tools

- **ZodiPy** (Provides Model(25 * u.micron) and the evaluate(skycoord, obspos=...) mechanism that resolves the observer position and computes line-of-sight zodiacal emission) — https://github.com/Cosmoglobe/zodipy
- **astropy** (Supplies SkyCoord, Time, and units used to define galactic coordinates, obstimes, and the 25 micron wavelength; ZodiPy requires Astropy >= 5.0.1)
- **healpy** (Optional helper to create a SkyCoord grid of pointing directions for the observer)

## Examples

```
model = zodipy.Model(25 * u.micron); emission = model.evaluate(SkyCoord(40*u.deg, 60*u.deg, obstime=Time('2020-01-01'), frame='galactic'), obspos='mars')
```

## Evaluation signals

- obspos='mars' at 25 micron must reproduce the documented 8.36985535 MJy/sr (versus 25.08189292 MJy/sr for the default Earth-center observer) within a small numerical tolerance
- The documented galactic 40/60 deg point sequence at obstime 2020-01-01 must yield 27.52410841, 27.66572294, 27.81251906 MJy/sr
- Output intensities carry MJy/sr units consistent with the 25 micron DIRBE wavelength initialization
- Explicit cartesian obspos and the equivalent body-name obspos (same ephemeris position) should agree
- point_evaluations.csv must contain one row per evaluation with coordinates, obstime, observer position, computed value, and reference value

## Limitations

- ZodiPy implements the COBE DIRBE interplanetary dust model and the Planck extension, valid for 1.25–240 µm (infrared) and 30–857 GHz (microwave); emissions outside these ranges rely on extrapolation
- Observer positioning via body name depends on the solar system ephemeris, so results are tied to the installed ephemeris and Python >= 3.9 with Astropy >= 5.0.1
- Validation is anchored to a small set of documented point evaluations (e.g. 2020-01-01 and 2022-01-01 timestamps) rather than full timestreams

## Evidence

- [abstract] abstract: "We present ZodiPy, a modern and easy-to-use Python package for modeling the zodiacal emission seen by an arbitrary Solar System observer"
- [methods] methods: "# Initialize a zodiacal light model at a wavelength/frequency or over a bandpass
model = zodipy.Model(25 * u.micron)"
- [intro] intro: "ZodiPy is a Python package for zodiacal light simulations."
- [methods] methods-requirements: "- [Astropy](https://www.astropy.org) (>= 5.0.1)"
- [methods] install: "Installing ZodiPy is as simple as:

```
pip install zodipy
"
