---
name: timestream-residual-comparison-across-software
description: Use when a new package reimplements an established instrument-forward-model
  tool and you must verify equivalence by producing the same timestreams for identical
  pointings/times and quantifying residuals.
when_to_use_negative:
- The goal is binned sky maps rather than time-ordered timestream comparisons (the
  article treats map-binning as a separate workflow).
- No legacy reference software or reference timestreams exist for the instrument,
  so there is nothing to compute residuals against.
- The new package targets a different instrument or dust model than the reference
  tool covers (ZodiPy implements the COBE DIRBE interplanetary dust model and Planck
  extension, 1.25–240 µm and 30–857 GHz).
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: ZodiPy
  role: Python package used to simulate the zodiacal emission timestreams being validated,
    via zodipy.Model and model.evaluate(skycoord)
  repo: https://github.com/Cosmoglobe/zodipy
- name: astropy
  role: Provides SkyCoord (frame='galactic', obstime) to specify scan coordinates
    and astropy.units (MJy/sr, micron) for quantities
  repo: https://www.astropy.org
- name: COBE DIRBE Zodiacal Light Prediction Software
  role: Legacy reference tool producing the DIRBE timestreams against which ZodiPy
    output is differenced
- name: healpy
  role: Alternative (with Astropy-healpix) for creating the SkyCoord directions over
    which the model is evaluated
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202244133
    title: 'COSMOGLOBE: Simulating zodiacal emission with ZodiPy'
  source_content_hash: sha256:052ea41aa685d886a1fc94dee13be6070cb23c4da86d505a616c3e0d8c070111
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# timestream-residual-comparison-across-software

## Summary

Cross-validate two independent software implementations by simulating the same instrumental timestreams with each (here, ZodiPy vs the COBE DIRBE Zodiacal Light Prediction Software for zodiacal emission) and comparing per-sample residuals.

## When to use

Use when a new package reimplements an established instrument-forward-model tool and you must verify equivalence by producing the same timestreams for identical pointings/times and quantifying residuals.

## When NOT to use

- The goal is binned sky maps rather than time-ordered timestream comparisons (the article treats map-binning as a separate workflow).
- No legacy reference software or reference timestreams exist for the instrument, so there is nothing to compute residuals against.
- The new package targets a different instrument or dust model than the reference tool covers (ZodiPy implements the COBE DIRBE interplanetary dust model and Planck extension, 1.25–240 µm and 30–857 GHz).

## Inputs

- Line-of-sight directions and observation times forming a DIRBE-like scan (Astropy SkyCoord, frame='galactic', obstime set)
- Photometric band wavelength or bandpass (e.g. 25 micron) for model initialization
- Reference timestreams from the COBE DIRBE Zodiacal Light Prediction Software for the same pointings/times

## Outputs

- Simulated zodiacal emission timestream in MJy/sr (zodipy_dirbe_timestream.fits)
- Per-sample residual and fractional-difference comparison table (dirbe_timestream_comparison.csv)

## How to apply

Install the new package (e.g. 'pip install zodipy' with its Astropy dependency) and build a scan of line-of-sight directions and observation times packaged into an Astropy SkyCoord with obstime set and frame='galactic', mirroring the package's SkyCoord-based interface. Initialize the forward model at the reference instrument's photometric band (e.g. zodipy.Model(25 * u.micron) for a DIRBE band) and evaluate it over the scan with model.evaluate(skycoord) to obtain simulated timestreams in MJy/sr. Run the legacy reference software (COBE DIRBE Zodiacal Light Prediction Software) for the same pointings and times to produce reference timestreams. Compute per-sample residuals (simulated minus reference) and fractional differences into a comparison table; agreement of the ZodiPy timestreams with the DIRBE ZLP timestreams, as reported in the paper, is the acceptance criterion. Save the simulated timestream as a FITS file (zodipy_dirbe_timestream.fits) and the residual table as CSV (dirbe_timestream_comparison.csv).

## Related tools

- **ZodiPy** (Python package used to simulate the zodiacal emission timestreams being validated, via zodipy.Model and model.evaluate(skycoord)) — https://github.com/Cosmoglobe/zodipy
- **astropy** (Provides SkyCoord (frame='galactic', obstime) to specify scan coordinates and astropy.units (MJy/sr, micron) for quantities) — https://www.astropy.org
- **COBE DIRBE Zodiacal Light Prediction Software** (Legacy reference tool producing the DIRBE timestreams against which ZodiPy output is differenced)
- **healpy** (Alternative (with Astropy-healpix) for creating the SkyCoord directions over which the model is evaluated)

## Examples

```
pip install zodipy; model = zodipy.Model(25 * u.micron); timestream = model.evaluate(SkyCoord(lon, lat, obstime=obstimes, frame='galactic'))
```

## Evaluation signals

- The residual table dirbe_timestream_comparison.csv exists and contains both simulated-minus-reference and fractional-difference fields for every scan sample.
- The simulated timestream zodipy_dirbe_timestream.fits is written and its values carry MJy/sr units consistent with the DIRBE ZLP reference output.
- Per-sample residuals between ZodiPy and DIRBE ZLP timestreams are consistent with the agreement reported in the paper for the same pointing/times.
- The SkyCoord uses frame='galactic' with obstime set, matching the package's SkyCoord-based interface, and the model was initialized at a DIRBE band (e.g. 25 micron).

## Limitations

- Validation is limited to the DIRBE interplanetary dust model and the Planck extension; predictions cover 1.25–240 µm and 30–857 GHz with extrapolation to other frequencies/wavelengths.
- Agreement was demonstrated against the DIRBE Zodiacal Light Prediction Software for DIRBE-like scans; other instruments or dust models are not covered by this comparison.
- Ruff and pyOpenSci review artifacts in the repo context indicate an actively developed package; API details may change between versions.

## Evidence

- [abstract] ZodiPy timestreams agree with DIRBE ZLP: "these agree with corresponding timestreams produced with the DIRBE Zodiacal Light Prediction Software"
- [methods] SkyCoord evaluation workflow: "skycoord = SkyCoord(lon, lat, obstime=obstimes, frame="galactic")"
- [methods] model initialization at band: "model = zodipy.Model(25 * u.micron)"
- [intro] package scope: "ZodiPy is a Python package for zodiacal light simulations."
- [intro] Astropy affiliation: "As an Astropy-affiliated package, ZodiPy is highly integrated with the Astropy ecosystem"
