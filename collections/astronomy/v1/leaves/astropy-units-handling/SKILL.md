---
name: astropy-units-handling
description: 'Use when building inputs for ZodiPy evaluations: any wavelength/bandpass,
  SkyCoord lon/lat, or obstime that feeds model.evaluate or grid_number_density must
  be expressed as Astropy Quantities/Time objects to get MJy/sr intensities.'
when_to_use_negative:
- Working with raw unitless NumPy arrays outside the Astropy ecosystem — attaching
  ad-hoc manual unit conversions instead of Quantities defeats the astropy.units integration
  ZodiPy relies on.
- Modeling zodiacal emission at wavelengths/frequencies outside ZodiPy's supported
  DIRBE ranges (1.25–240 µm infrared, 30–857 GHz microwave) — unit handling cannot
  compensate for an unsupported model band.
tools:
- name: ZodiPy
  role: Consumes Astropy Quantities for model initialization (wavelength/bandpass)
    and evaluates zodiacal emission in MJy/sr
  repo: github.com/Cosmoglobe/zodipy
- name: astropy
  role: Provides astropy.units, SkyCoord, and Time used to construct unit-correct
    model and coordinate inputs (>= 5.0.1 required)
- name: jplephem
  role: Solar system ephemeris backing obspos body-name lookups such as obspos='mars'
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202244133
    title: 'COSMOGLOBE: Simulating zodiacal emission with ZodiPy'
  source_content_hash: sha256:8151ab2d170f7b7f50285417cfe361e439faa934bf2e56d80aa0ee556f1a83f0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - ZodiPy
  - astropy
  - jplephem
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# astropy-units-handling

## Summary

Correctly attach and propagate Astropy units when driving ZodiPy: wavelengths/frequencies passed to zodipy.Model must be astropy.units Quantities (e.g. 25 * u.micron), and SkyCoord longitudes/latitudes carry deg units, so that evaluated zodiacal emission comes back in physically meaningful MJy/sr.

## When to use

Use when building inputs for ZodiPy evaluations: any wavelength/bandpass, SkyCoord lon/lat, or obstime that feeds model.evaluate or grid_number_density must be expressed as Astropy Quantities/Time objects to get MJy/sr intensities.

## When NOT to use

- Working with raw unitless NumPy arrays outside the Astropy ecosystem — attaching ad-hoc manual unit conversions instead of Quantities defeats the astropy.units integration ZodiPy relies on.
- Modeling zodiacal emission at wavelengths/frequencies outside ZodiPy's supported DIRBE ranges (1.25–240 µm infrared, 30–857 GHz microwave) — unit handling cannot compensate for an unsupported model band.

## Inputs

- astropy.units Quantity specifying the model wavelength/frequency or bandpass (e.g. 25 * u.micron)
- astropy.coordinates.SkyCoord with galactic lon/lat in deg units and obstime as astropy Time
- obspos keyword: solar system body name (e.g. 'mars') or heliocentric ecliptic cartesian position

## Outputs

- Zodiacal emission intensities in MJy/sr per SkyCoord point (from model.evaluate)
- Table of coordinates, obstime, observer position, computed vs. reference values (e.g. point_evaluations.csv)

## How to apply

Wrap every physical quantity in an Astropy unit before it touches ZodiPy: initialize the model with a Quantity such as zodipy.Model(25 * u.micron) (wavelength within the DIRBE-covered 1.25–240 µm infrared range or 30–857 GHz microwave band), rather than a bare float. Build the coordinate array with galactic longitudes/latitudes carrying deg units, e.g. lon = [10, 10.1, 10.2] * u.deg, and attach obstime as an astropy Time (e.g. 2020-01-01 or 2022-01-01 timestamps) inside a SkyCoord with frame='galactic'. When repositioning the observer, pass obspos as a solar system body name such as 'mars', which ZodiPy resolves through the solar system ephemeris (jplephem) — the returned intensity is in MJy/sr and should be compared against documented reference values (25.08189292 MJy/sr for the Earth-center observer, 8.36985535 MJy/sr for Mars at 25 micron) within a small numerical tolerance. The rationale is that ZodiPy is Astropy-affiliated and highly integrated with astropy.units, so unit-correct inputs are what guarantee the documented MJy/sr outputs and enable reproducible point-evaluation tables like point_evaluations.csv.

## Related tools

- **ZodiPy** (Consumes Astropy Quantities for model initialization (wavelength/bandpass) and evaluates zodiacal emission in MJy/sr) — github.com/Cosmoglobe/zodipy
- **astropy** (Provides astropy.units, SkyCoord, and Time used to construct unit-correct model and coordinate inputs (>= 5.0.1 required))
- **jplephem** (Solar system ephemeris backing obspos body-name lookups such as obspos='mars')

## Examples

```
model = zodipy.Model(25 * u.micron); skycoord = SkyCoord(lon, lat, obstime=obstimes, frame="galactic"); emission = model.evaluate(skycoord)
```

## Evaluation signals

- zodipy.Model raises/behaves incorrectly if the wavelength is not a Quantity — the model must be initialized with something like 25 * u.micron, not a bare number.
- model.evaluate returns intensities with MJy/sr units; the point evaluation at galactic 40/60 deg with obstime 2020-01-01 must reproduce the documented 27.52410841/27.66572294/27.81251906 sequence within a small numerical tolerance.
- With obspos='mars' at 25 micron the computed intensity must match 8.36985535 MJy/sr (versus 25.08189292 MJy/sr for the default Earth-center observer).
- SkyCoord inputs carry deg units and an obstime; a unitless coordinate array or missing obstime is a sign the skill was not applied.

## Limitations

- ZodiPy's supported wavelength/frequency coverage is bounded by the DIRBE model (1.25–240 µm) and the Planck extension (30–857 GHz); unit-correct inputs outside these ranges still cannot produce valid predictions.
- Observer repositioning via obspos body names depends on the solar system ephemeris (jplephem), so results depend on ephemeris availability and the supplied obstime.

## Evidence

- [intro] Astropy integration for units: "As an Astropy-affiliated package, ZodiPy is highly integrated with the Astropy ecosystem, particularly with the [`astropy.units`]"
- [methods] Model initialization with a Quantity: "# Initialize a zodiacal light model at a wavelength/frequency or over a bandpass
model = zodipy.Model(25 * u.micron)"
- [methods] Unit-correct SkyCoord construction: "# Use Astropy's `SkyCoord` object to specify coordinates
lon = [10, 10.1, 10.2] * u.deg"
