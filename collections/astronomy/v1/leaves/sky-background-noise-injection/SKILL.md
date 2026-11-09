---
name: sky-background-noise-injection
description: Use when converting an SKIRT IFU-like ideal data cube (skirt_view_(ViewID)_total.fits)
  into a mock observational image and the output must include per-filter sky background
  noise consistent with a named survey's throughputs.
when_to_use_negative:
- When the ideal IFU-like data cube itself is the desired product (dataCubes/ directory)
  and observational realism should not be added
- When the input is already real observational data rather than a simulated data cube
  — noise injection only makes sense on synthetic images
- When only PSF convolution is required and background realism is intentionally excluded
  (conf['includeBkg_CSST'] set to False)
tools:
- name: GalaxyGenius
  role: Python package whose PostProcess.runPostprocess() stage integrates the cube
    over filter throughputs and adds sky background noise per band
  repo: https://github.com/xczhou-astro/galaxyGenius
- name: SKIRT
  role: Produces the IFU-like ideal data cube that the noise-injection step operates
    on during postprocessing
- name: numpy
  role: Loads filter throughput data and applies the per-band background-noise arrays
    to bandpass images
- name: astropy
  role: Reads and writes the FITS data cube and the resulting galaxy_images.fits with
    one band per extension
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202554287
    title: 'GalaxyGenius: Mock galaxy image generator for various telescopes from
      hydrodynamical simulations'
  source_content_hash: sha256:1deba99a73bdf70a9c75665233e5052a45c2d2e38e1501e6e52cbbdaf5117809
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xczhou-astro/galaxyGenius
  tools:
  - CSST
  - GalaxyGenius
  - Python
  - SKIRT
  - astropy
  - numpy
  - photutils
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# sky-background-noise-injection

## Summary

Adds realistic sky-background noise to mock bandpass images during GalaxyGenius's postprocessing stage, computing noise from each filter's throughput curve and the sky emission curve (or from limiting magnitudes) so SKIRT-generated ideal data cubes resemble real observations from telescopes such as CSST.

## When to use

Use when converting an SKIRT IFU-like ideal data cube (skirt_view_(ViewID)_total.fits) into a mock observational image and the output must include per-filter sky background noise consistent with a named survey's throughputs.

## When NOT to use

- When the ideal IFU-like data cube itself is the desired product (dataCubes/ directory) and observational realism should not be added
- When the input is already real observational data rather than a simulated data cube — noise injection only makes sense on synthetic images
- When only PSF convolution is required and background realism is intentionally excluded (conf['includeBkg_CSST'] set to False)

## Inputs

- SKIRT-generated ideal data cube FITS file (skirt_view_(ViewID)_total.fits) for the target subhalo
- Survey configuration file (e.g. config_CSST.toml) with the background-noise toggle (includeBkg_CSST)
- Filter throughput files in Data/filters/(survey) as two-column plain-text .fil files (wavelength in angstrom, throughput)
- Sky emission curve used with each filter's throughput
- Optionally, survey limiting magnitudes from which to derive the noise level

## Outputs

- Mock bandpass images with injected sky background noise, written to mock_(survey)/Subhalo_(ID)/galaxy_images.fits with one image per band as a FITS extension

## How to apply

After the DataGeneration stage produces the ideal data cube, run the GalaxyGenius PostProcess stage with the survey config (e.g. config_CSST.toml) and the background-noise toggle enabled (conf['includeBkg_CSST']). For each requested bandpass, integrate the cube spectrum over the .fil filter throughput, then add sky background noise computed from that filter's throughput and the sky emission curve, or derive the noise level from limiting magnitudes when sky curves are unavailable. The helper notebook Notebooks/calc_sky_bkg.ipynb (Bkg/calc_sky_bkg.ipynb) documents the instrumental-noise calculation and can be adapted for new surveys. Results are written per survey into mock_CSST/Subhalo_(ID)/galaxy_images.fits with one band per FITS extension.

## Related tools

- **GalaxyGenius** (Python package whose PostProcess.runPostprocess() stage integrates the cube over filter throughputs and adds sky background noise per band) — https://github.com/xczhou-astro/galaxyGenius
- **SKIRT** (Produces the IFU-like ideal data cube that the noise-injection step operates on during postprocessing)
- **numpy** (Loads filter throughput data and applies the per-band background-noise arrays to bandpass images)
- **astropy** (Reads and writes the FITS data cube and the resulting galaxy_images.fits with one band per extension)

## Examples

```
from galaxyGenius.postprocess import PostProcess; postprocess = PostProcess(subhaloID=ID); postprocess.runPostprocess()  # with includeBkg enabled in config_CSST.toml
```

## Evaluation signals

- galaxy_images.fits exists in mock_(survey)/Subhalo_(ID)/ and contains one image extension per requested survey band (e.g. CSST bands)
- Pixel values in each band extension are finite and non-negative after noise injection
- Per-band background level matches the noise expected from the filter's throughput convolved with the sky emission curve, or the published limiting magnitude
- Bands generated with includeBkg disabled show systematically lower background scatter than bands with noise injected, confirming the toggle acts on the correct extension
- The calc_sky_bkg.ipynb noise calculation reproduces the pipeline's derived noise level for the same filter and sky emission curve

## Limitations

- Noise level can alternatively be derived from limiting magnitudes, which is an approximation when the full sky emission curve is not used
- For Euclid, PSF files are not provided (only filters), so Euclid mocks rely on separate PSF handling even though filter throughputs are available
- The Data directory of filters, PSFs and sky curves must be correctly set via GALAXYGENIUS_DATA_DIR, and filters/PSFs must exist in the correct directories and formats before running

## Evidence

- [results] Sky background noises are calculated based on the throughput of each filter and sky emission curve.: "Sky background noises are calculated based on the throughput of each filter and sky emission curve."
- [readme] helper notebook calc_sky_bkg: "We provide a helper notebook for calculation of instrumental noises in [Notebooks/calc_sky_bkg.ipynb]"
- [results] limiting-magnitude noise: "Add support for deriving noise level from limiting magnitudes."
- [abstract] mock observation module applies instrumental effects: "the mock observation module takes the ideal data cube and applies the throughputs of aiming telescopes, while also incorporating the relevant instrumental effects"
