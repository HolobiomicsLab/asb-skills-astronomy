---
name: sky-background-noise-from-limiting-magnitudes
description: Use when generating mock observational images for surveys like CSST,
  HST, JWST, Roman, HSC (or Euclid, filters only) and you must add per-band background
  noise to SKIRT ideal data cubes, or derive noise levels from survey limiting magnitudes.
when_to_use_negative:
- Input is already a real observed image — noise synthesis only applies to mock images
  built from simulation particles via SKIRT ideal data cubes
- Euclid mock observations requiring PSF convolution — the Euclid PSF files cannot
  be accessed, so only filter throughput (and thus filter-based noise scaling) can
  be applied
- A priori noise is already characterized in external data — use preprocess.inputs/preprocess.inputSubhaloParticleFile-style
  custom pipelines rather than the default noise model
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: GalaxyGenius
  role: Python package whose PostProcess.runPostprocess() stage applies filter throughputs
    and injects sky background noise computed from throughput and the sky emission
    curve; supports deriving noise from limiting magnitudes
  repo: github.com/xczhou-astro/galaxyGenius
- name: Bkg/calc_sky_bkg.ipynb
  role: Helper notebook for calculation of instrumental and sky background noises
    per survey filter
  repo: github.com/xczhou-astro/galaxyGenius/blob/main/Bkg/calc_sky_bkg.ipynb
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202554287
    title: 'GalaxyGenius: Mock galaxy image generator for various telescopes from
      hydrodynamical simulations'
  source_content_hash: sha256:e11896e058c21bd732123d20f85dbb1ca7d03ba96814a412bacd9f8e55cf2cf4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bkg/calc_sky_bkg.ipynb
  - CSST
  - Euclid
  - GalaxyGenius
  - HSC
  - HST
  - JWST
  - Python
  - Roman
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# sky-background-noise-from-limiting-magnitudes

## Summary

Derive realistic sky background and instrumental noise levels for mock survey images, either from each filter's throughput curve combined with a sky emission curve, or directly from survey limiting magnitudes, as implemented in the GalaxyGenius post-processing stage.

## When to use

Use when generating mock observational images for surveys like CSST, HST, JWST, Roman, HSC (or Euclid, filters only) and you must add per-band background noise to SKIRT ideal data cubes, or derive noise levels from survey limiting magnitudes.

## When NOT to use

- Input is already a real observed image — noise synthesis only applies to mock images built from simulation particles via SKIRT ideal data cubes
- Euclid mock observations requiring PSF convolution — the Euclid PSF files cannot be accessed, so only filter throughput (and thus filter-based noise scaling) can be applied
- A priori noise is already characterized in external data — use preprocess.inputs/preprocess.inputSubhaloParticleFile-style custom pipelines rather than the default noise model

## Inputs

- Filter throughput files (.fil plain text, columns: wavelength in angstrom, throughput) from Data/filters/(survey)
- Sky emission curve for the survey site (space or ground-based)
- Survey limiting magnitudes (alternative basis for deriving noise level)
- SKIRT ideal data cube (skirt_view_(ViewID)_total.fits) from DataGeneration.runSKIRT with numPackets=1e6 in DustEmission mode
- Per-survey config.toml options (e.g. includeBkg_CSST, includePSF_CSST)

## Outputs

- Per-band sky background noise levels for each survey filter
- Noisy bandpass images written to FITS extensions of mock_(survey)/Subhalo_(ID)/galaxy_images.fits
- All-band composite plots all_bands_view_(ViewID).png with realistic background

## How to apply

After the SKIRT radiative transfer stage produces the IFU-like ideal data cube (skirt_view_(ViewID)_total.fits), compute sky background noise per bandpass using each survey's filter throughput file (.fil plain-text, wavelength in angstrom plus throughput columns) convolved with a sky emission curve. Where a sky emission curve is impractical, derive the noise level directly from the survey's published limiting magnitudes, a feature GalaxyGenius added explicitly for this purpose. Use the helper notebook Bkg/calc_sky_bkg.ipynb to compute the instrumental/sky noises for the target survey, then enable the noise via per-survey config options (e.g. includeBkg_CSST) in the workspace config.toml before running PostProcess.runPostprocess(). Noise is injected during postprocessing alongside PSF convolution, so the noise model must match the survey-specific pixel scale and filter set.

## Related tools

- **GalaxyGenius** (Python package whose PostProcess.runPostprocess() stage applies filter throughputs and injects sky background noise computed from throughput and the sky emission curve; supports deriving noise from limiting magnitudes) — github.com/xczhou-astro/galaxyGenius
- **Bkg/calc_sky_bkg.ipynb** (Helper notebook for calculation of instrumental and sky background noises per survey filter) — github.com/xczhou-astro/galaxyGenius/blob/main/Bkg/calc_sky_bkg.ipynb

## Examples

```
python init.py --workspace=workspace --surveys="CSST,HSC"
```

## Evaluation signals

- mock_(survey)/Subhalo_(ID) directories exist for all configured surveys and galaxy_images.fits contains bandpass images in its FITS extensions
- Per-band noise values are consistent with the filter's .fil throughput integrated against the sky emission curve (or with the published limiting magnitude when derived from limiting magnitudes)
- Toggling includeBkg_(survey) in config changes the pixel statistics of the output bandpass images (background level appears/disappears)
- Sky noise differs appropriately between ground-based (e.g. HSC, CSST) and space-based (e.g. JWST, Roman) surveys per their sky emission curves
- Re-running the helper notebook Bkg/calc_sky_bkg.ipynb reproduces the noise levels used by PostProcess

## Limitations

- Noise derivation from the sky emission curve requires the throughput and sky emission data for each survey; where these are unavailable, limiting magnitudes must be used instead
- Web-based API mode (requests=true) is slower and depends on network speed, which affects the end-to-end pipeline that includes noise injection
- Euclid is limited to filters only, since its PSF files cannot be accessed, constraining the realism of its mock observations

## Evidence

- [results] Sky background noises are calculated based on the throughput of each filter and sky emission curve.: "Sky background noises are calculated based on the throughput of each filter and sky emission curve."
- [results] Noise from limiting magnitudes: "Add support for deriving noise level from limiting magnitudes."
- [readme] Helper notebook for instrumental noises: "We provide a helper notebook for calculation of instrumental noises in [Notebooks/calc_sky_bkg.ipynb](https://github.com/xczhou-astro/galaxyGenius/blob/main/Bkg/calc_sky_bkg.ipynb)."
