---
name: astropy-sky-coordinate-observer-evaluation
description: Use when you need pointwise or timestream zodiacal emission intensities
  at specific sky coordinates and obstimes from a ZodiPy model (dirbe, planck13/15/18,
  rrm-experimental, odegard), e.g. to reproduce DIRBE or Planck predictions.
when_to_use_negative:
- The task is simulating binned full-sky zodiacal emission maps (e.g. DIRBE binned
  maps) rather than pointwise SkyCoord evaluation
- The requested wavelengths/frequencies lie outside the 1.25–240 µm or 30–857 GHz
  coverage of the built-in models and extrapolation is not acceptable
- The observer geometry requires a dust model not registered in ZodiPy
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: ZodiPy
  role: Provides zodipy.Model, which is initialized at a wavelength/frequency or bandpass
    and evaluated on a SkyCoord to return zodiacal emission intensity in MJy/sr
  repo: https://github.com/Cosmoglobe/zodipy
- name: astropy
  role: Supplies SkyCoord (galactic frame with obstime) and astropy.time.Time used
    as inputs to model.evaluate; ZodiPy is Astropy-affiliated and integrated with
    astropy.units
  repo: https://www.astropy.org
- name: healpy
  role: Alternative to Astropy-healpix for creating the SkyCoord input used in the
    evaluation
  repo: https://healpy.readthedocs.io/en/latest/
- name: Astropy-healpix
  role: Alternative to healpy for creating the SkyCoord input used in the evaluation
  repo: https://astropy-healpix.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202244133
    title: 'COSMOGLOBE: Simulating zodiacal emission with ZodiPy'
  source_content_hash: sha256:df58aafa970aa3813fd8d2fea21ab9642a51dc5a6a1fa9d2aaffe8347588b303
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
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# astropy-sky-coordinate-observer-evaluation

## Summary

Evaluate a ZodiPy zodiacal light model at specified sky positions and observation times by building an astropy SkyCoord in the galactic frame and calling model.evaluate, yielding intensity predictions in MJy/sr for an arbitrary Solar System observer.

## When to use

Use when you need pointwise or timestream zodiacal emission intensities at specific sky coordinates and obstimes from a ZodiPy model (dirbe, planck13/15/18, rrm-experimental, odegard), e.g. to reproduce DIRBE or Planck predictions.

## When NOT to use

- The task is simulating binned full-sky zodiacal emission maps (e.g. DIRBE binned maps) rather than pointwise SkyCoord evaluation
- The requested wavelengths/frequencies lie outside the 1.25–240 µm or 30–857 GHz coverage of the built-in models and extrapolation is not acceptable
- The observer geometry requires a dust model not registered in ZodiPy

## Inputs

- astropy SkyCoord in the galactic frame with longitudes (e.g. [10, 10.1, 10.2] deg), latitudes (e.g. [90, 89, 88] deg), and obstimes (e.g. 2022-01-01 12:00–12:02) via astropy.time.Time
- Registered ZodiPy model name (dirbe, rrm-experimental, planck13, planck15, planck18, odegard)
- Initialization wavelength/frequency or bandpass (wavelengths within 1.25–240 µm, frequencies within 30–857 GHz)

## Outputs

- Zodiacal emission intensity values in MJy/sr per SkyCoord position and obstime
- Coverage/prediction table (columns: model, wavelength/frequency, emission), e.g. saved as model_emission_coverage.csv

## How to apply

Install ZodiPy via pip into a Python >= 3.9 environment with Astropy >= 5.0.1, NumPy, jplephem, and SciPy. Instantiate zodipy.Model for the registered model of interest, initialized either at a single wavelength/frequency (e.g. 25 micron) or over a bandpass, staying within the model's valid 1.25–240 µm infrared or 30–857 GHz microwave coverage. Build an astropy SkyCoord in frame='galactic' from longitudes, latitudes, and astropy.time.Time obstimes for the observer. Call model.evaluate(skycoord) and record the returned intensities in MJy/sr, assembling results across model–wavelength combinations into a coverage table. Validate that every model instantiates and returns finite MJy/sr intensities at each sampled wavelength/frequency, flagging any combination that fails.

## Related tools

- **ZodiPy** (Provides zodipy.Model, which is initialized at a wavelength/frequency or bandpass and evaluated on a SkyCoord to return zodiacal emission intensity in MJy/sr) — https://github.com/Cosmoglobe/zodipy
- **astropy** (Supplies SkyCoord (galactic frame with obstime) and astropy.time.Time used as inputs to model.evaluate; ZodiPy is Astropy-affiliated and integrated with astropy.units) — https://www.astropy.org
- **healpy** (Alternative to Astropy-healpix for creating the SkyCoord input used in the evaluation) — https://healpy.readthedocs.io/en/latest/
- **Astropy-healpix** (Alternative to healpy for creating the SkyCoord input used in the evaluation) — https://astropy-healpix.readthedocs.io/en/latest/

## Examples

```
from zodipy import Model; from astropy.coordinates import SkyCoord; model = Model(25 * u.micron); intensity_mJy_sr = model.evaluate(SkyCoord(lon, lat, obstime=obstimes, frame='galactic'))
```

## Evaluation signals

- All six registered models (dirbe, rrm-experimental, planck13, planck15, planck18, odegard) instantiate successfully and return intensities at every sampled wavelength/frequency
- Returned intensities carry MJy/sr units as expected from the model output
- Predicted emission values are produced at every model–wavelength/frequency combination in the coverage table; failures to instantiate are explicitly flagged
- For DIRBE wavelengths, simulated timestreams agree with those produced by the DIRBE Zodiacal Light Prediction Software

## Limitations

- Built-in models are bounded: infrared predictions cover 1.25–240 µm and microwave predictions 30–857 GHz; other frequencies/wavelengths rely on extrapolation
- ZodiPy supports a fixed set of registered models, so custom interplanetary dust models are not covered by this workflow
- Model evaluation requires ephemeris support (jplephem dependency) and Python >= 3.9 with Astropy >= 5.0.1

## Evidence

- [abstract] Coverage spans all registered models: "allows for zodiacal emission predictions at infrared wavelengths in the 1.25–240 µm range and at microwave frequencies in the 30–857 GHz range"
- [methods] SkyCoord workflow: "# Use Astropy's `SkyCoord` object to specify coordinates
lon = [10, 10.1, 10.2] * u.deg ... skycoord = SkyCoord(lon, lat, obstime=obstimes, frame="galactic")"
- [methods] Model initialization: "# Initialize a zodiacal light model at a wavelength/frequency or over a bandpass
model = zodipy.Model(25 * u.micron)"
- [methods] Installation and dependencies: "- [Astropy](https://www.astropy.org) (>= 5.0.1)"
- [intro] Purpose of the package: "Its purpose is to provide the astrophysics and cosmology communities with an easy-to-use and accessible interface to existing zodiacal light models in Python"
- [abstract] Validation against DIRBE software: "these agree with corresponding timestreams produced with the DIRBE Zodiacal Light Prediction Software"
