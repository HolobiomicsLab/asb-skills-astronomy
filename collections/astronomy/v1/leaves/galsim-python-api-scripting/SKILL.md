---
name: galsim-python-api-scripting
description: Use when you need to generate a synthetic image of a galaxy convolved
  with a PSF plus Gaussian pixel noise using the GalSim Python API, e.g. to replicate
  the demo1.py tutorial or build a simple forward-model simulation.
when_to_use_negative:
- You only want to specify the simulation declaratively — use the parallel demo1.yaml
  config-file route with the galsim executable instead of writing Python
- You need chromatic objects drawn through a bandpass, real COSMOS RealGalaxy data,
  or atmospheric PSF screens — demo1.py is a single-galaxy parametric Gaussian demo
- The noise model required is not simple Gaussian pixel noise (e.g. correlated noise
  needing whitening/symmetrizing) and the GaussianNoise approach does not apply
edam_operation: http://edamontology.org/operation_0337
tools:
- name: GalSim
  role: Provides the Python API (GSObject profiles, drawImage, image.addNoise with
    GaussianNoise) used to simulate the PSF-convolved, noise-added galaxy image
  repo: https://github.com/GalSim-developers/GalSim
- name: Python
  role: User interface language in which demo1.py is written and run from the examples/
    directory
  repo: https://github.com/GalSim-developers/GalSim
- name: numpy
  role: Numeric types used for Image inputs/outputs in the GalSim Python layer
  repo: https://github.com/GalSim-developers/GalSim
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.02.002
    title: 'GalSim: The modular galaxy image simulation toolkit'
  source_content_hash: sha256:460ddecc4e537a26b14f0c993ab6ea7474e82f430afbbcf69f47837c447e099c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/GalSim-developers/GalSim
  tools:
  - GalSim
  - Python
  - numpy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# galsim-python-api-scripting

## Summary

Reproduces GalSim's demo1.py tutorial workflow in Python: build a parametric galaxy profile (sigma 2 arcsec, flux 1.e5), convolve with a Gaussian PSF (sigma 1 arcsec) at 0.2 arcsec/pixel, draw the image, add Gaussian noise via galsim.GaussianNoise, and write a deterministic demo1.fits.

## When to use

Use when you need to generate a synthetic image of a galaxy convolved with a PSF plus Gaussian pixel noise using the GalSim Python API, e.g. to replicate the demo1.py tutorial or build a simple forward-model simulation.

## When NOT to use

- You only want to specify the simulation declaratively — use the parallel demo1.yaml config-file route with the galsim executable instead of writing Python
- You need chromatic objects drawn through a bandpass, real COSMOS RealGalaxy data, or atmospheric PSF screens — demo1.py is a single-galaxy parametric Gaussian demo
- The noise model required is not simple Gaussian pixel noise (e.g. correlated noise needing whitening/symmetrizing) and the GaussianNoise approach does not apply

## Inputs

- GalSim examples/demo1.py script from the GalSim-developers/GalSim repository
- Python environment with the galsim module (and numpy) installed
- demo1 parameter values: galaxy sigma 2 arcsec, flux 1.e5 counts; PSF sigma 1 arcsec; pixel_scale 0.2 arcsec/pixel; noise sigma 30

## Outputs

- demo1.fits — a FITS image of the PSF-convolved galaxy with Gaussian noise (expected grid size given pixel_scale 0.2 arcsec/pixel)
- Deterministic pixel values identical to the deposited reference demo output (fixed random seed)

## How to apply

Run the demo script from the GalSim examples/ directory (scripts are designed to be run there: 'python demo1.py') with the galsim module imported. Build the galaxy and PSF surface brightness profiles with the Python API using the demo1 parameter values — galaxy sigma 2 arcsec with flux 1.e5 counts, PSF sigma 1 arcsec, pixel_scale 0.2 arcsec/pixel — and draw the convolved image with drawImage. Add Gaussian noise via image.addNoise(galsim.GaussianNoise(sigma=30)), where sigma=30 is the standard deviation of counts in each pixel, using the fixed random seed baked into the script so the run is fully deterministic. Write the resulting image to disk in FITS format as demo1.fits in the output directory. Validate that demo1.fits exists, is a valid FITS image of the expected size, and that pixel values match the deposited reference demo output exactly.

## Related tools

- **GalSim** (Provides the Python API (GSObject profiles, drawImage, image.addNoise with GaussianNoise) used to simulate the PSF-convolved, noise-added galaxy image) — https://github.com/GalSim-developers/GalSim
- **Python** (User interface language in which demo1.py is written and run from the examples/ directory) — https://github.com/GalSim-developers/GalSim
- **numpy** (Numeric types used for Image inputs/outputs in the GalSim Python layer) — https://github.com/GalSim-developers/GalSim

## Examples

```
image.addNoise(galsim.GaussianNoise(sigma=30)); image.write('demo1.fits')
```

## Evaluation signals

- demo1.fits exists in the output directory and is a valid FITS image of the expected size (pixel_scale 0.2 arcsec/pixel grid)
- Image pixel values precisely match the deposited reference demo output — the fixed seed makes the run fully deterministic
- The noise was applied via the expected call pattern: image.addNoise(galsim.GaussianNoise(sigma=noise)) with sigma=30
- The drawn image is a single galaxy convolved with the specified Gaussian PSF (sigma 1 arcsec) with total flux consistent with 1.e5 counts
- Script completes when run as 'python demo1.py' from the examples/ directory without path errors

## Limitations

- Demo scripts must be run from the examples/ directory; some access files in subdirectories and would not work correctly from other locations
- The fixed random seed means results are deterministic — changing the seed changes pixel values and breaks exact-match validation against the reference output
- The public API is preserved only within a given major version number (e.g. 2.x.x), so very old script versions may not run unchanged on future major versions
- The demo uses simple parametric Gaussian models; it does not cover real-galaxy (COSMOS RealGalaxy) simulations, chromatic bandpass drawing, or atmospheric PSFs

## Evidence

- [results] Add Gaussian noise to the image: "image.addNoise(galsim.GaussianNoise(sigma=noise))"
- [results] Draw convolved galaxy image and write to disk: "We draw an image of a single galaxy convolved
with a PSF and write it to disk."
- [readme] Demo scripts live in examples/: [evidence span withheld — anchor status: external_doc]
- [intro] Python user interface: "The bulk of the calculations are carried out in C++, and the user interface is in Python."
