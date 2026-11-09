---
name: point-spread-function-convolution-mock-imaging
description: Use when generating mock observational galaxy images from radiative-transfer
  data cubes and need survey-realistic PSF blurring, filter throughput application,
  and sky background noise injection per band.
when_to_use_negative:
- The input is raw simulation particle data that still needs SKIRT radiative transfer
  — apply the PreProcess and DataGeneration stages first
- Euclid mock imaging requiring a convolved PSF — only filters are uploaded since
  the Euclid PSF files cannot be accessed
- The goal is pure radiative-transfer SED modelling without observation simulation
  — the PSF/instrumental/noise postprocessing is unnecessary
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3382
tools:
- name: GalaxyGenius
  role: Python package whose PostProcess.runPostprocess() stage applies survey throughputs,
    PSFs, instrumental effects, and sky background to the ideal data cube
  repo: https://github.com/xczhou-astro/galaxyGenius
- name: SKIRT
  role: Radiative transfer code producing the ideal data cubes that are subsequently
    PSF-convolved in postprocessing
- name: numpy
  role: Used to load survey PSF arrays via numpy.loadtxt or numpy.load for convolution
- name: illustris_python
  role: Extracts subhalo star and gas particles from TNG snapshots feeding the upstream
    data-cube stage
  repo: https://github.com/illustristng/illustris_python
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202554287
    title: 'GalaxyGenius: Mock galaxy image generator for various telescopes from
      hydrodynamical simulations'
  source_content_hash: sha256:5a02e3f690c0a75a8850cb8201c51cbdcd2d66722e8effa501a28704a3ec792b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xczhou-astro/galaxyGenius
  tools:
  - CSST
  - Euclid
  - GalaxyGenius
  - HSC
  - HST
  - JWST
  - Python
  - Roman
  - SKIRT
  - illustris_python
  - numpy
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# point-spread-function-convolution-mock-imaging

## Summary

Convolve ideal IFU-like SKIRT data cubes with per-survey point spread functions (stored as numpy arrays in Data/PSFs/(survey)) while applying filter throughputs, instrumental effects, and sky background noise, to produce realistic mock bandpass images for surveys such as CSST, Euclid, HST, JWST, Roman, and HSC from IllustrisTNG and EAGLE simulations.

## When to use

Use when generating mock observational galaxy images from radiative-transfer data cubes and need survey-realistic PSF blurring, filter throughput application, and sky background noise injection per band.

## When NOT to use

- The input is raw simulation particle data that still needs SKIRT radiative transfer — apply the PreProcess and DataGeneration stages first
- Euclid mock imaging requiring a convolved PSF — only filters are uploaded since the Euclid PSF files cannot be accessed
- The goal is pure radiative-transfer SED modelling without observation simulation — the PSF/instrumental/noise postprocessing is unnecessary

## Inputs

- IFU-like ideal data cube from SKIRT (skirt_view_(ViewID)_total.fits) per subhalo and view
- Filter throughput .fil plain-text files (wavelength in angstrom, throughput) from Data/filters/(survey)
- Survey PSFs as numpy arrays (loadable via numpy.loadtxt or numpy.load) from Data/PSFs/(survey)
- Sky emission curve and filter throughput for sky background noise calculation
- Per-survey config.toml flags (e.g. includePSF_CSST, includeBkg_CSST)

## Outputs

- Bandpass mock images stored in FITS extensions of mock_(survey)/Subhalo_(ID)/galaxy_images.fits
- All-band composite plots all_bands_view_(ViewID).png
- SEDs in galaxy_SEDs.fits

## How to apply

During the PostProcess stage of the GalaxyGenius pipeline, take the SKIRT-produced IFU-like ideal data cube (skirt_view_(ViewID)_total.fits) and apply each telescope's filter throughputs read from plain-text .fil files (two columns: wavelength in angstrom and throughput) located in Data/filters/(survey). Convolve the resulting bandpass images with the survey PSF, stored as a numpy array loadable via numpy.loadtxt or numpy.load in Data/PSFs/(survey), and add instrumental effects and sky background noise computed from the filter throughput and sky emission curve (helper notebook Notebooks/calc_sky_bkg.ipynb). Enable per-survey effects via config flags such as includePSF_CSST and includeBkg_CSST. Note that Euclid PSF files are unavailable, so for Euclid only filter throughputs are applied. Verify that PSF files exist in the correct directory and format before running, since the pipeline requires filters and PSFs present in correct directories and formats.

## Related tools

- **GalaxyGenius** (Python package whose PostProcess.runPostprocess() stage applies survey throughputs, PSFs, instrumental effects, and sky background to the ideal data cube) — https://github.com/xczhou-astro/galaxyGenius
- **SKIRT** (Radiative transfer code producing the ideal data cubes that are subsequently PSF-convolved in postprocessing)
- **numpy** (Used to load survey PSF arrays via numpy.loadtxt or numpy.load for convolution)
- **illustris_python** (Extracts subhalo star and gas particles from TNG snapshots feeding the upstream data-cube stage) — https://github.com/illustristng/illustris_python

## Examples

```
postprocess = PostProcess(subhaloID=ID); postprocess.runPostprocess()
```

## Evaluation signals

- mock_(survey)/Subhalo_(ID) directories exist for all six surveys (CSST, Euclid, HST, JWST, Roman, HSC) after postprocessing
- galaxy_images.fits contains bandpass images in its FITS extensions for each survey's filters
- Convolved images match the paper's showcased CSST, Euclid, JWST, and HSC band and RGB composite outputs (e.g. make_rgb from Astropy)
- Filters and PSFs exist in correct directories (Data/filters/(survey), Data/PSFs/(survey)) and formats (.fil plain text; numpy arrays) before running
- Sky background noise is consistent with values derived from each filter's throughput and the sky emission curve, or from limiting magnitudes

## Limitations

- Euclid PSF files are not provided (cannot be accessed), so only filter throughputs are applied for Euclid
- Sky background/noise generation speed depends on the network when using the TNG Web API rather than local snapshots
- PSF application depends on per-survey config flags; forgetting includePSF_(survey) or includeBkg_(survey) silently omits those instrumental effects

## Evidence

- [abstract] mock observation applies throughputs, PSFs, instrumental effects, and background noise: "the mock observation module takes the ideal data cube and applies the throughputs of aiming telescopes, while also incorporating the relevant instrumental effects, point spread functions (PSFs)"
- [readme] PSF file format and loading: "PSFs are recommended to be in `numpy.array`, and can be opened by `numpy.loadtxt` or `numpy.load`."
- [results] Euclid PSFs missing: "For Euclid, we only upload the filters, since the PSF files cannot be accessed."
- [results] Sky background noise derivation: "Sky background noises are calculated based on the throughput of each filter and sky emission curve."
- [results] Output FITS structure with bandpass images: "`./dataCubes` includes the IFU-like raw dataCubes and SEDs of **SubhaloID** generated by SKIRT"
