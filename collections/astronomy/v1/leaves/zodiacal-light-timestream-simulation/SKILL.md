---
name: zodiacal-light-timestream-simulation
description: Use when a task requires forward-simulated zodiacal light timestreams
  or binned zodiacal emission maps for a Solar System observer (e.g. DIRBE-like pointings/times),
  or when validating a zodiacal light model against DIRBE ZLP reference output.
when_to_use_negative:
- The task needs foreground subtraction or removal of zodiacal emission from real
  data rather than forward simulation of predicted emission
- The observer geometry or observing epoch cannot be expressed as pointings + obstimes
  in a SkyCoord (e.g. no Solar System observer context)
- The target wavelength/frequency lies outside ZodiPy's implemented model ranges (1.25–240
  µm, 30–857 GHz) and extrapolation is not acceptable
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3898
tools:
- name: ZodiPy
  role: Python package used to model zodiacal emission for an arbitrary Solar System
    observer; provides Model initialization and evaluate(skycoord) to produce the
    timestream
  repo: https://github.com/Cosmoglobe/zodipy
- name: astropy
  role: Provides SkyCoord with obstime and frame='galactic' to define scan coordinates,
    and astropy.units for the band wavelength (e.g. 25 * u.micron)
  repo: https://www.astropy.org
- name: healpy
  role: Can be used together with Astropy-healpix to create the SkyCoord of line-of-sight
    directions
- name: COBE DIRBE Zodiacal Light Prediction Software
  role: Generates the reference DIRBE timestreams against which the ZodiPy-simulated
    timestreams are compared
- name: Python
  role: Implementation language for the simulation and residual-comparison workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202244133
    title: 'COSMOGLOBE: Simulating zodiacal emission with ZodiPy'
  source_content_hash: sha256:b24bd53696b03bce418521b8016eadf4093c9d71c53869f29321223cdd59667a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Cosmoglobe/zodipy
  tools:
  - COBE DIRBE Zodiacal Light Prediction Software
  - Python
  - ZodiPy
  - astropy
  - healpy
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3898
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# zodiacal-light-timestream-simulation

## Summary

Simulate zodiacal emission timestreams for an experiment like COBE DIRBE using the ZodiPy Python package, and validate them against reference timestreams from the DIRBE Zodiacal Light Prediction Software.

## When to use

Use when a task requires forward-simulated zodiacal light timestreams or binned zodiacal emission maps for a Solar System observer (e.g. DIRBE-like pointings/times), or when validating a zodiacal light model against DIRBE ZLP reference output.

## When NOT to use

- The task needs foreground subtraction or removal of zodiacal emission from real data rather than forward simulation of predicted emission
- The observer geometry or observing epoch cannot be expressed as pointings + obstimes in a SkyCoord (e.g. no Solar System observer context)
- The target wavelength/frequency lies outside ZodiPy's implemented model ranges (1.25–240 µm, 30–857 GHz) and extrapolation is not acceptable

## Inputs

- Line-of-sight sky directions for the observer scan (galactic lon/lat) packaged as an Astropy SkyCoord with obstime set
- Observation times (obstimes) matching the scan samples
- Photometric band wavelength/frequency or bandpass (e.g. 25 micron; ZodiPy supports 1.25–240 µm and 30–857 GHz)
- Reference DIRBE timestreams from the COBE DIRBE Zodiacal Light Prediction Software (and optionally DIRBE calibrated individual observations, CIO)

## Outputs

- Simulated zodiacal emission timestream (MJy/sr) per sample, saved as zodipy_dirbe_timestream.fits
- Residual comparison table (simulated minus reference and fractional difference per sample), saved as dirbe_timestream_comparison.csv
- Optionally binned maps of the zodiacal emission predicted for DIRBE

## How to apply

Install ZodiPy with 'pip install zodipy' (Astropy-affiliated, requires Astropy >= 5.0.1). Build a DIRBE-like scan by defining line-of-sight directions (e.g. galactic lon/lat lists) and observation times, packaged into an Astropy SkyCoord with obstime set and frame='galactic'; healpy or Astropy-healpix can be used to create the SkyCoord. Initialize the model at a DIRBE photometric band wavelength with zodipy.Model(25 * u.micron) — ZodiPy implements the COBE DIRBE interplanetary dust model and the Planck extension covering 1.25–240 µm and 30–857 GHz. Evaluate the model over the scan with model.evaluate(skycoord) to obtain the timestream in MJy/sr. Produce reference timestreams with the COBE DIRBE Zodiacal Light Prediction Software for the same pointings/times, then compute per-sample residuals (simulated minus reference, plus fractional differences) into a comparison table; the paper's validation is that ZodiPy timestreams agree with the corresponding DIRBE ZLP timestreams.

## Related tools

- **ZodiPy** (Python package used to model zodiacal emission for an arbitrary Solar System observer; provides Model initialization and evaluate(skycoord) to produce the timestream) — https://github.com/Cosmoglobe/zodipy
- **astropy** (Provides SkyCoord with obstime and frame='galactic' to define scan coordinates, and astropy.units for the band wavelength (e.g. 25 * u.micron)) — https://www.astropy.org
- **healpy** (Can be used together with Astropy-healpix to create the SkyCoord of line-of-sight directions)
- **COBE DIRBE Zodiacal Light Prediction Software** (Generates the reference DIRBE timestreams against which the ZodiPy-simulated timestreams are compared)
- **Python** (Implementation language for the simulation and residual-comparison workflow)

## Examples

```
from zodipy import Model; from astropy.coordinates import SkyCoord; import astropy.units as u; model = Model(25 * u.micron); emission = model.evaluate(SkyCoord(lon, lat, obstime=obstimes, frame='galactic'))
```

## Evaluation signals

- The ZodiPy timestream values carry MJy/sr units consistent with the DIRBE ZLP reference output
- dirbe_timestream_comparison.csv exists and contains the residual fields (simulated-minus-reference and fractional difference per sample)
- zodipy_dirbe_timestream.fits exists and its per-sample residuals show agreement with the DIRBE Zodiacal Light Prediction Software timestreams, as reported in the paper
- The SkyCoord was constructed with obstime set and frame='galactic', mirroring ZodiPy's SkyCoord-based interface
- The model was initialized at a valid wavelength/frequency or bandpass within the supported 1.25–240 µm / 30–857 GHz ranges

## Limitations

- ZodiPy covers the COBE DIRBE interplanetary dust model and the Planck extension only over 1.25–240 µm and 30–857 GHz, with extrapolation needed for other frequencies/wavelengths
- Validation is against the DIRBE Zodiacal Light Prediction Software; agreement with other instruments or real DIRBE calibrated individual observations (CIO) must be established separately
- ZodiPy requires Astropy >= 5.0.1 as a dependency

## Evidence

- [abstract] ZodiPy implements the COBE DIRBE interplanetary dust model and the Planck extension, covering 1.25–240 µm and 30–857 GHz, with extrapolation to other frequencies/wavelengths.: "allows for zodiacal emission predictions at infrared wavelengths in the 1.25–240 µm range and at microwave frequencies in the 30–857 GHz range"
- [methods] Workflow installs ZodiPy via pip and uses it with Astropy.: "Installing ZodiPy is as simple as:

```
pip install zodipy
"
- [methods] Model initialization at a wavelength and evaluation over a SkyCoord scan.: "# Initialize a zodiacal light model at a wavelength/frequency or over a bandpass
model = zodipy.Model(25 * u.micron)"
- [methods] SkyCoord construction with obstime and galactic frame, then model evaluation.: "skycoord = SkyCoord(lon, lat, obstime=obstimes, frame="galactic")"
- [methods] Astropy integration and dependency version.: "- [Astropy](https://www.astropy.org) (>= 5.0.1)"
- [methods] ZodiPy is an Astropy-affiliated package for simulating zodiacal light intensity for arbitrary observers.: "ZodiPy is an [Astropy-affiliated](https://www.astropy.org/affiliated/) package for simulating zodiacal light in intensity for arbitrary solar system observers."
- [abstract] Reference comparison with DIRBE timestreams.: "these agree with corresponding timestreams produced with the DIRBE Zodiacal Light Prediction Software"
- [intro] healpy or Astropy-healpix can be used to build the SkyCoord.: "We can use [healpy](https://healpy.readthedocs.io/en/latest/) or [Astropy-healpix](https://astropy-healpix.readthedocs.io/en/latest/) to create a [`SkyCoord`]"
