---
name: observer-pointing-coordinate-handling
description: Use when a simulation or binning step requires observer pointings expressed
  as astropy SkyCoord objects with obstime and Galactic-frame coordinates, e.g. to
  feed ZodiPy's zodipy.Model or bin a timestream into a HEALPix map.
when_to_use_negative:
- The pointings are static sky directions with no obstime (e.g. a fixed template map)
  — ZodiPy needs obstime to place the observer in the Solar System.
- Coordinates are in a non-Galactic frame without conversion — the article's workflow
  specifies SkyCoord objects with Galactic-frame coordinates.
- The goal is modeling a foreground other than zodiacal emission; ZodiPy only implements
  zodiacal light models (COBE DIRBE IPD model and the Planck extension).
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3391
tools:
- name: ZodiPy
  role: Evaluates the zodiacal light model at the SkyCoord-specified pointings to
    produce the observer timestream
  repo: https://github.com/Cosmoglobe/zodipy
- name: astropy
  role: Provides the SkyCoord object and astropy.units integration used to specify
    coordinates and obstime
- name: healpy
  role: Can be used to create pixel-center coordinates for the SkyCoord when sampling
    over a HEALPix grid
  repo: https://healpy.readthedocs.io/en/latest/
- name: Astropy-healpix
  role: Alternative to healpy for creating a SkyCoord from HEALPix pixelization
  repo: https://astropy-healpix.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202244133
    title: 'COSMOGLOBE: Simulating zodiacal emission with ZodiPy'
  source_content_hash: sha256:05e6106c8b4c2039ba6289cf11756d9090050c7bfd4e34f66956d1604a47c76f
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# observer-pointing-coordinate-handling

## Summary

Constructs Astropy SkyCoord objects carrying obstime and Galactic-frame coordinates for a Solar System observer's pointings, so that ZodiPy can evaluate zodiacal emission timestreams along the actual observing geometry.

## When to use

Use when a simulation or binning step requires observer pointings expressed as astropy SkyCoord objects with obstime and Galactic-frame coordinates, e.g. to feed ZodiPy's zodipy.Model or bin a timestream into a HEALPix map.

## When NOT to use

- The pointings are static sky directions with no obstime (e.g. a fixed template map) — ZodiPy needs obstime to place the observer in the Solar System.
- Coordinates are in a non-Galactic frame without conversion — the article's workflow specifies SkyCoord objects with Galactic-frame coordinates.
- The goal is modeling a foreground other than zodiacal emission; ZodiPy only implements zodiacal light models (COBE DIRBE IPD model and the Planck extension).

## Inputs

- Observer pointing longitudes and latitudes (e.g. deg arrays in Galactic frame)
- Per-sample observing times (astropy time objects for the obstime keyword)
- Target wavelength/frequency or bandpass (e.g. 25 micron)
- HEALPix pixel centers from healpy or Astropy-healpix (optional, for map-space sampling)

## Outputs

- Astropy SkyCoord object with obstime and frame='galactic'
- Zodiacal emission timestream evaluated at the observer pointings (via zodipy.Model)

## How to apply

Express the observer's pointings as longitude/latitude arrays in the Galactic frame, attach per-sample observing times via the SkyCoord obstime keyword, and construct a single SkyCoord object using healpy or Astropy-healpix pixelization utilities for pixel centers if needed. The obstime is essential because ZodiPy uses it to locate the observer in the Solar System when evaluating the interplanetary dust model at wavelengths in the 1.25–240 µm range or frequencies in the 30–857 GHz range. Pass the resulting SkyCoord to zodipy.Model evaluation to obtain the zodiacal emission timestream sampled at those pointings. Rationale: ZodiPy is Astropy-affiliated and highly integrated with astropy.units, so coordinate/time handling in the Astropy ecosystem guarantees frame consistency with downstream HEALPix binning.

## Related tools

- **ZodiPy** (Evaluates the zodiacal light model at the SkyCoord-specified pointings to produce the observer timestream) — https://github.com/Cosmoglobe/zodipy
- **astropy** (Provides the SkyCoord object and astropy.units integration used to specify coordinates and obstime)
- **healpy** (Can be used to create pixel-center coordinates for the SkyCoord when sampling over a HEALPix grid) — https://healpy.readthedocs.io/en/latest/
- **Astropy-healpix** (Alternative to healpy for creating a SkyCoord from HEALPix pixelization) — https://astropy-healpix.readthedocs.io/en/latest/

## Examples

```
from astropy.coordinates import SkyCoord; skycoord = SkyCoord(lon, lat, obstime=obstimes, frame="galactic")
```

## Evaluation signals

- The SkyCoord object carries both obstime and frame='galactic'; omitting obstime should raise an error when evaluated with zodipy.Model since observer ephemeris is required.
- Unit consistency: longitudes/latitudes constructed with astropy.units (e.g. u.deg) and the model initialized with a compatible unit (e.g. 25 * u.micron).
- Timestream length must match the number of pointings/samples in the input SkyCoord.
- For DIRBE validation, the ZodiPy timestream should agree with corresponding timestreams produced with the DIRBE Zodiacal Light Prediction Software.
- Emission values should fall within the physical intensity range expected for the chosen wavelength band (1.25–240 µm or 30–857 GHz coverage).

## Limitations

- ZodiPy covers zodiacal emission predictions only in the 1.25–240 µm infrared range and 30–857 GHz microwave range, with extrapolation to other frequencies/wavelengths; outside this, accuracy is not validated here.
- Coordinates without attached obstime cannot be used, since the zodiacal emission seen depends on the observer's Solar System position.
- Requires Astropy >= 5.0.1 for the SkyCoord/units API used.

## Evidence

- [methods] skycoord with obstime in galactic frame: "# Use Astropy's `SkyCoord` object to specify coordinates
lon = [10, 10.1, 10.2] * u.deg ... skycoord = SkyCoord(lon, lat, obstime=obstimes, frame="galactic")"
- [intro] healpy or Astropy-healpix for pixel coordinates: "We can use [healpy](https://healpy.readthedocs.io/en/latest/) or [Astropy-healpix](https://astropy-healpix.readthedocs.io/en/latest/) to create a [`SkyCoord`]"
- [intro] Astropy ecosystem integration: "As an Astropy-affiliated package, ZodiPy is highly integrated with the Astropy ecosystem, particularly with the [`astropy.units`]"
- [abstract] arbitrary observer support: "We present ZodiPy, a modern and easy-to-use Python package for modeling the zodiacal emission seen by an arbitrary Solar System observer"
