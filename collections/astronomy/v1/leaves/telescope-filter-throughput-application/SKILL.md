---
name: telescope-filter-throughput-application
description: Use when an ideal SKIRT data cube (skirt_view_(ViewID)_total.fits) must
  be converted into survey-specific mock images, or when custom filter throughput
  (.fil) and PSF (numpy array) data need to be folded into a mock observation.
when_to_use_negative:
- The SKIRT radiative transfer stage has not yet been run — PostProcess requires the
  ideal data cube from dataCubes/ as input
- 'You need Euclid PSF-convolved images: Euclid PSF files are not provided (cannot
  be accessed), only its filters'
- The goal is simulating instrumental detector artifacts beyond throughputs, PSFs,
  and sky background noise — GalaxyGenius postprocessing does not model these
edam_operation: http://edamontology.org/operation_0337
tools:
- name: GalaxyGenius
  role: PostProcess module applies telescope throughputs, instrumental effects, PSFs,
    and sky background noise to the ideal data cube; per-survey config options control
    which effects are included
  repo: https://github.com/xczhou-astro/galaxyGenius
- name: SKIRT
  role: Produces the upstream IFU-like ideal data cube (DustEmission mode, numPackets=1e6)
    that the filter/PSF postprocessing consumes
  repo: https://skirt.ugent.be/root/_installation_guide.html
- name: illustris_python
  role: Provides particle inputs extracted from local TNG snapshots that feed the
    pipeline whose postprocessing stage applies filters and PSFs
  repo: https://github.com/illustristng/illustris_python
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202554287
    title: 'GalaxyGenius: Mock galaxy image generator for various telescopes from
      hydrodynamical simulations'
  source_content_hash: sha256:216eff16211afd9c007453508ec36452eee7122fbce672b038a1e29a0f525f63
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# telescope-filter-throughput-application

## Summary

Apply survey-specific filter throughputs, PSFs, instrumental effects, and sky background noise to an IFU-like ideal data cube (produced by SKIRT) to generate mock observational bandpass images for CSST, Euclid, HST, JWST, Roman, and HSC from IllustrisTNG/EAGLE galaxies.

## When to use

Use when an ideal SKIRT data cube (skirt_view_(ViewID)_total.fits) must be converted into survey-specific mock images, or when custom filter throughput (.fil) and PSF (numpy array) data need to be folded into a mock observation.

## When NOT to use

- The SKIRT radiative transfer stage has not yet been run — PostProcess requires the ideal data cube from dataCubes/ as input
- You need Euclid PSF-convolved images: Euclid PSF files are not provided (cannot be accessed), only its filters
- The goal is simulating instrumental detector artifacts beyond throughputs, PSFs, and sky background noise — GalaxyGenius postprocessing does not model these

## Inputs

- IFU-like ideal data cube from SKIRT (dataCubes/, skirt_view_(ViewID)_total.fits)
- Filter throughput .fil plain-text files (wavelength in angstrom + throughput columns) per survey
- Survey PSF numpy arrays (loadable with numpy.loadtxt or numpy.load)
- Sky emission curve and per-survey config.toml options (e.g. includeBkg_CSST, includePSF_CSST)
- Subhalo particle inputs from IllustrisTNG snapshots or EAGLE RefL0100N1504

## Outputs

- mock_(survey)/Subhalo_(ID)/galaxy_images.fits with bandpass images in FITS extensions per survey band
- galaxy_SEDs.fits containing per-band SEDs
- All-band composite plots (all_bands_view_(ViewID).png) and Astropy make_rgb RGB composites

## How to apply

Run the GalaxyGenius PostProcess stage (postprocess.runPostprocess()) on the SKIRT ideal data cube after setting GALAXYGENIUS_DATA_DIR so Data/filters/(survey) and Data/PSFs/(survey) are found. Filter throughputs must be plain-text .fil files with two columns (wavelength in angstrom, throughput); PSFs must be numpy arrays loadable via numpy.loadtxt or numpy.load. Sky background noise is computed from each filter's throughput curve combined with a sky emission curve (helper notebook Notebooks/calc_sky_bkg.ipynb), and noise can alternatively be derived from limiting magnitudes. Configure per-survey options in config_(survey).toml files created by init.py (e.g. includeBkg_CSST, includePSF_CSST); note that Euclid supports filters only because its PSF files cannot be accessed, so apply filters without PSF convolution for Euclid. The resulting bandpass images are written into FITS extensions of mock_(survey)/Subhalo_(ID)/galaxy_images.fits, with SEDs in galaxy_SEDs.fits and all-band composite plots all_bands_view_(ViewID).png.

## Related tools

- **GalaxyGenius** (PostProcess module applies telescope throughputs, instrumental effects, PSFs, and sky background noise to the ideal data cube; per-survey config options control which effects are included) — https://github.com/xczhou-astro/galaxyGenius
- **SKIRT** (Produces the upstream IFU-like ideal data cube (DustEmission mode, numPackets=1e6) that the filter/PSF postprocessing consumes) — https://skirt.ugent.be/root/_installation_guide.html
- **illustris_python** (Provides particle inputs extracted from local TNG snapshots that feed the pipeline whose postprocessing stage applies filters and PSFs) — https://github.com/illustristng/illustris_python

## Examples

```
python init.py --workspace=workspace --surveys="CSST,HSC"; export GALAXYGENIUS_DATA_DIR=/path/to/Data; postprocess.runPostprocess()
```

## Evaluation signals

- mock_(survey)/Subhalo_(ID) directories exist for all requested surveys (CSST, Euclid, HST, JWST, Roman, HSC)
- galaxy_images.fits contains one bandpass image per survey filter in its FITS extensions
- Output matches the paper's showcased CSST, Euclid, JWST, and HSC band and RGB composite images (e.g. CSST g, r, z RGB for EAGLE subhalos 20695120 and 21109761)
- Euclid output contains filter-applied images only, since PSFs are absent by design
- Sky background noise levels are consistent with calculations from filter throughput and the sky emission curve (Notebooks/calc_sky_bkg.ipynb)

## Limitations

- Euclid PSF files are not provided (cannot be accessed), so Euclid mock images are filter-only without PSF convolution
- Sky background computation depends on the throughput of each filter and the supplied sky emission curve; wrong units in .fil files (wavelength must be angstrom) will corrupt results
- If the TNG Web API mode is used instead of local snapshots, generation speed is slower and depends on network speed
- Data directory must be set via GALAXYGENIUS_DATA_DIR or filters/PSFs will not be located

## Evidence

- [abstract] Mock observation applies throughputs, PSFs, background: "the mock observation module takes the ideal data cube and applies the throughputs of aiming telescopes, while also incorporating the relevant instrumental effects, point spread functions (PSFs), and"
- [results] Filter .fil format: "The format of filters are recommended to be plain text file with extension `.fil`, including two columns: wavelength (in angstrom) and throughputs."
- [results] PSF format: "PSFs are recommended to be in `numpy.array`, and can be opened by `numpy.loadtxt` or `numpy.load`."
- [results] Euclid PSF missing: "For Euclid, we only upload the filters, since the PSF files cannot be accessed."
- [results] Sky background from throughput and sky emission: "Sky background noises are calculated based on the throughput of each filter and sky emission curve."
- [readme] PostProcess invocation: "postprocess = PostProcess(subhaloID=ID) # initialize PostProcess class
    postprocess.runPostprocess() # run postprocessing"
- [readme] Environment variable for Data directory: "`Data` directory should be set in environment, named `GALAXYGENIUS_DATA_DIR`"
