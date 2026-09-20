---
name: multi-band-galaxy-image-visualization
description: Use when hydrodynamical simulation particle data (IllustrisTNG or EAGLE
  subhalos) must be turned into survey-realistic bandpass images or RGB composites,
  e.g. to build deep-learning training sets or validate observational pipelines.
when_to_use_negative:
- The input is already an observational image or a reduced survey data product rather
  than simulation particle data — GalaxyGenius starts from star and gas particle properties
  of a selected subhalo.
- Only a single-band, noise-free synthetic image is needed without survey-specific
  throughputs, PSFs, or sky background — the mock observation module's per-survey
  configuration would be unnecessary overhead.
- The survey of interest requires PSF convolution for Euclid — PSF files for Euclid
  are not provided (cannot be accessed), so only filters are applied for that survey.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3379
tools:
- name: GalaxyGenius
  role: Python package orchestrating preprocessing, SKIRT radiative transfer, and
    mock observation postprocessing for six surveys
  repo: github.com/xczhou-astro/galaxyGenius
- name: SKIRT
  role: Radiative transfer code executed by DataGeneration.runSKIRT in DustEmission
    mode to assign SEDs per particle and produce the IFU-like ideal data cube
- name: illustris_python
  role: Used by the pipeline to extract subhalos and particles from local TNG snapshot
    directories
  repo: github.com/illustristng/illustris_python
- name: Astropy
  role: Provides make_rgb with default parameters for creating RGB composite images
    from bandpass images
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202554287
    title: 'GalaxyGenius: Mock galaxy image generator for various telescopes from
      hydrodynamical simulations'
  source_content_hash: sha256:1ef4bcc23fe9d9ea53d70162681b0ad997b6ac0b76de7cbc42b129279345a382
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Astropy
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3379
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# multi-band-galaxy-image-visualization

## Summary

A GalaxyGenius-based skill for rendering mock multi-band galaxy images (UV to IR) from hydrodynamical simulation subhalos as they would appear to space and ground-based surveys (CSST, Euclid, HST, JWST, Roman, HSC), by post-processing SKIRT ideal data cubes with filter throughputs, PSFs, and instrumental noise.

## When to use

Use when hydrodynamical simulation particle data (IllustrisTNG or EAGLE subhalos) must be turned into survey-realistic bandpass images or RGB composites, e.g. to build deep-learning training sets or validate observational pipelines.

## When NOT to use

- The input is already an observational image or a reduced survey data product rather than simulation particle data — GalaxyGenius starts from star and gas particle properties of a selected subhalo.
- Only a single-band, noise-free synthetic image is needed without survey-specific throughputs, PSFs, or sky background — the mock observation module's per-survey configuration would be unnecessary overhead.
- The survey of interest requires PSF convolution for Euclid — PSF files for Euclid are not provided (cannot be accessed), so only filters are applied for that survey.

## Inputs

- IllustrisTNG (e.g. TNG100 snapshot 94 subhalo 31 h5 cutout) or EAGLE (RefL0100N1504 snapshot 27, z=0.101) subhalo particle data
- Filter throughput files (.fil plain text: wavelength in angstrom + throughput columns) in Data/filters/(survey)
- PSF files (numpy arrays loadable via numpy.loadtxt or numpy.load) in Data/PSFs/(survey)
- Workspace config.toml and per-survey config_*.toml files (e.g. includeBkg_CSST, includePSF_CSST flags)
- SKIRT9 installation path

## Outputs

- mock_(survey)/Subhalo_(ID)/galaxy_images.fits with bandpass images in FITS extensions per survey (CSST, Euclid, HST, JWST, Roman, HSC)
- galaxy_SEDs.fits with per-subhalo SEDs
- All-band composite plots all_bands_view_(ViewID).png
- RGB composite images (e.g. Astropy make_rgb from CSST g, r, z bands)
- IFU-like ideal data cubes skirt_view_(ViewID)_total.fits and dataCubes/ from SKIRT

## How to apply

Initialize a workspace with `python init.py --workspace=workspace --surveys="CSST,HSC"` to create config.toml and per-survey config files, and set GALAXYGENIUS_DATA_DIR to the Data directory containing filters and PSFs. For each subhalo, run PreProcess (extract star/gas particles and create the .ski file), DataGeneration.runSKIRT (DustEmission mode, numPackets=1e6) to produce the IFU-like ideal data cube, then PostProcess.runPostprocess() to apply each survey's filter throughputs, PSFs, instrumental effects, and sky background noise computed from filter throughput and the sky emission curve. For EAGLE data, load particles via preprocess.inputParticles with custom starFunction/starFormingFunction/dustFunction and preprocess.createFile. Store bandpass images in the FITS extensions of mock_(survey)/Subhalo_(ID)/galaxy_images.fits and SEDs in galaxy_SEDs.fits, and render all-band composites as all_bands_view_(ViewID).png; RGB images are created using default parameters for make_rgb in Astropy. Note that for Euclid only filters are applied since the PSF files cannot be accessed.

## Related tools

- **GalaxyGenius** (Python package orchestrating preprocessing, SKIRT radiative transfer, and mock observation postprocessing for six surveys) — github.com/xczhou-astro/galaxyGenius
- **SKIRT** (Radiative transfer code executed by DataGeneration.runSKIRT in DustEmission mode to assign SEDs per particle and produce the IFU-like ideal data cube)
- **illustris_python** (Used by the pipeline to extract subhalos and particles from local TNG snapshot directories) — github.com/illustristng/illustris_python
- **Astropy** (Provides make_rgb with default parameters for creating RGB composite images from bandpass images)

## Examples

```
python init.py --workspace=workspace --surveys="CSST,HSC"; python run.py  # run.py calls preprocess.prepare(arguments), dataGeneration.runSKIRT(skirtPATh='path/to/skirt'), postprocess.runPostprocess()
```

## Evaluation signals

- mock_(survey)/Subhalo_(ID) directories exist for all six surveys (CSST, Euclid, HST, JWST, Roman, HSC) after runPostprocess()
- galaxy_images.fits contains bandpass images in its FITS extensions, one per requested filter band
- All-band composite plots (all_bands_view_(ViewID).png) render with consistent per-band images matching the paper's showcased CSST, Euclid, JWST, and HSC band and RGB outputs
- Filters and PSFs exist in correct directories and formats (.fil two-column text; numpy-loadable PSF arrays) before running galaxyGenius
- Sky background noise values match calculations based on each filter's throughput and the sky emission curve (verifiable via the calc_sky_bkg.ipynb helper notebook)

## Limitations

- For Euclid, only filters are uploaded since the PSF files cannot be accessed, so Euclid images lack PSF convolution.
- When using the TNG Web API mode (requests=true with apiKey), generation speed is slower and depends on network speed; the pipeline adds check and retry features for API failures.
- Users must ensure filters and PSFs exist in the correct directories and formats before running galaxyGenius; wrong formats or missing Data/ paths (GALAXYGENIUS_DATA_DIR unset) will break postprocessing.

## Evidence

- [abstract] Six-survey coverage of mock images from hydrodynamical simulations: "we created a series of mock images of galaxies based on the IllustrisTNG and EAGLE simulations for both space and ground-based surveys, spanning ultraviolet (UV) to infrared (IR)"
- [abstract] Mock observation module applies throughputs, instrumental effects, PSFs, and noise: "the mock observation module takes the ideal data cube and applies the throughputs of aiming telescopes, while also incorporating the relevant instrumental effects, point spread functions (PSFs), and"
- [readme] Filter file format specification: "The format of filters are recommended to be plain text file with extension `.fil`, including two columns: wavelength (in angstrom) and throughputs."
- [results] Sky background noise derivation: "Sky background noises are calculated based on the throughput of each filter and sky emission curve."
- [readme] Euclid PSF limitation: "For Euclid, we only upload the filters, since the PSF files cannot be accessed."
