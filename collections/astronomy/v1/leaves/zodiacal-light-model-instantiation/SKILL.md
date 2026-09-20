---
name: zodiacal-light-model-instantiation
description: Use when simulating zodiacal emission or interplanetary dust number densities
  for an arbitrary Solar System observer, and a supported zodiacal light model must
  be initialized at a defined infrared wavelength (1.25–240 µm) or microwave frequency
  (30–857 GHz).
when_to_use_negative:
- The task concerns foreground components other than zodiacal light (e.g. galactic
  dust or CMB foreground models) — ZodiPy implements only interplanetary dust models
- A self-contained reproduction of legacy DIRBE timestreams is required without cross-validation
  — ZodiPy reproduces the DIRBE Zodiacal Light Prediction Software but is a separate
  implementation
- The required wavelength/frequency lies far outside 1.25–240 µm or 30–857 GHz and
  extrapolation errors are unacceptable for the analysis
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: ZodiPy
  role: Provides the zodipy.Model interface to instantiate supported zodiacal light
    models at a wavelength/frequency or bandpass and evaluate emission for an arbitrary
    Solar System observer
  repo: https://github.com/Cosmoglobe/zodipy
- name: astropy
  role: Supplies SkyCoord, astropy.time.Time, and astropy.units used to specify observer
    coordinates, obstimes, and the model wavelength/frequency (e.g. 25 * u.micron)
  repo: https://www.astropy.org
- name: healpy
  role: Alternative to Astropy-healpix for creating the SkyCoord pixelization over
    which the model is evaluated
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202244133
    title: 'COSMOGLOBE: Simulating zodiacal emission with ZodiPy'
  source_content_hash: sha256:90dabb20fd6ff4a70915b0b0ea7ee284dcdeee86eee46ef00fbdafd4e0ec3e4c
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

# zodiacal-light-model-instantiation

## Summary

Instantiate a supported zodiacal light model in ZodiPy (e.g. dirbe, planck13/15/18, rrm-experimental, odegard) at a specific wavelength, frequency, or bandpass to enable zodiacal emission predictions with an Astropy-integrated Python package.

## When to use

Use when simulating zodiacal emission or interplanetary dust number densities for an arbitrary Solar System observer, and a supported zodiacal light model must be initialized at a defined infrared wavelength (1.25–240 µm) or microwave frequency (30–857 GHz).

## When NOT to use

- The task concerns foreground components other than zodiacal light (e.g. galactic dust or CMB foreground models) — ZodiPy implements only interplanetary dust models
- A self-contained reproduction of legacy DIRBE timestreams is required without cross-validation — ZodiPy reproduces the DIRBE Zodiacal Light Prediction Software but is a separate implementation
- The required wavelength/frequency lies far outside 1.25–240 µm or 30–857 GHz and extrapolation errors are unacceptable for the analysis

## Inputs

- Chosen zodiacal light model name (dirbe, rrm-experimental, planck13, planck15, planck18, odegard)
- Wavelength or frequency as an astropy Quantity (e.g. 25 micron) or a bandpass definition
- Observer sky coordinates as an astropy SkyCoord with obstime (astropy.time.Time) and frame (e.g. galactic)
- Python >= 3.9 environment with zodipy, Astropy >= 5.0.1, NumPy, jplephem, SciPy

## Outputs

- Zodiacal emission intensities in MJy/sr for the requested observer points (model.evaluate output)
- Component number-density distributions on a spatial grid (grid_number_density output, e.g. grid_number_density.png)
- Coverage/prediction tables (e.g. model_emission_coverage.csv) aggregating model–wavelength/frequency–emission values

## How to apply

Install ZodiPy via pip (Python >= 3.9), which pulls in Astropy >= 5.0.1, NumPy, jplephem, and SciPy. Choose a registered model (dirbe, rrm-experimental, planck13, planck15, planck18, odegard) and instantiate zodipy.Model either at a single wavelength/frequency (e.g. 25 micron via astropy.units) or over a bandpass; wavelengths must lie in the 1.25–240 µm infrared range or 30–857 GHz microwave range, with extrapolation available outside those ranges. Build observer pointing with an astropy SkyCoord in a chosen frame (e.g. galactic) carrying obstime from astropy.time.Time, then call model.evaluate(skycoord) to get intensities in MJy/sr. For spatial inspection of the underlying IPD distribution rather than emission, use the grid_number_density routine on the instantiated model. Rationale: ZodiPy is Astropy-affiliated and its Model interface is the documented entry point for both single-band and bandpass initialization of the DIRBE IPD model and its Planck extensions.

## Related tools

- **ZodiPy** (Provides the zodipy.Model interface to instantiate supported zodiacal light models at a wavelength/frequency or bandpass and evaluate emission for an arbitrary Solar System observer) — https://github.com/Cosmoglobe/zodipy
- **astropy** (Supplies SkyCoord, astropy.time.Time, and astropy.units used to specify observer coordinates, obstimes, and the model wavelength/frequency (e.g. 25 * u.micron)) — https://www.astropy.org
- **healpy** (Alternative to Astropy-healpix for creating the SkyCoord pixelization over which the model is evaluated)

## Examples

```
import zodipy; from astropy import units as u; model = zodipy.Model(25 * u.micron); intensity = model.evaluate(skycoord)
```

## Evaluation signals

- All six registered models (dirbe, rrm-experimental, planck13, planck15, planck18, odegard) instantiate without error at the sampled wavelength/frequency
- model.evaluate returns intensity arrays with units of MJy/sr at every sampled point
- Simulated DIRBE timestreams agree with those produced by the DIRBE Zodiacal Light Prediction Software (the article's validation benchmark)
- grid_number_density.png is a valid PNG depicting the IPD component number-density distribution over the spatial grid
- Any model/sampling-point combination that fails to instantiate is flagged in the coverage table

## Limitations

- Built-in models cover only 1.25–240 µm (infrared) and 30–857 GHz (microwave); predictions outside these ranges rely on extrapolation
- Number of supported models is fixed to the registered set; unsupported model names will fail at Model initialization
- Evaluations depend on JPL ephemerides (jplephem dependency) for observer/Earth position, so ephemeris availability affects instantiation

## Evidence

- [methods] Model instantiation at wavelength/frequency: "# Initialize a zodiacal light model at a wavelength/frequency or over a bandpass
model = zodipy.Model(25 * u.micron)"
- [intro] Astropy integration and unit handling: "As an Astropy-affiliated package, ZodiPy is highly integrated with the Astropy ecosystem, particularly with the [`astropy.units`]"
- [abstract] Coverage ranges of built-in models: "allows for zodiacal emission predictions at infrared wavelengths in the 1.25–240 µm range and at microwave frequencies in the 30–857 GHz range"
- [abstract] Validation against DIRBE software: "these agree with corresponding timestreams produced with the DIRBE Zodiacal Light Prediction Software"
- [methods] Installation method: "Installing ZodiPy is as simple as:

```
pip install zodipy
```"
