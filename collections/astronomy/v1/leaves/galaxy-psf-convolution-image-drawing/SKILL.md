---
name: galaxy-psf-convolution-image-drawing
description: Use when the task is to generate a synthetic galaxy image with a known
  surface brightness profile (e.g. Gaussian sigma 2 arcsec, flux 1.e5) convolved with
  a PSF (sigma 1 arcsec), on a 0.2 arcsec/pixel grid, with specified pixel noise.
when_to_use_negative:
- The galaxy should come from real HST/COSMOS observations rather than a simple parametric
  model — use GalSim's RealGalaxy with the real_galaxy_catalog (e.g. sample '23.5'
  or '25.2') instead
- The target is chromatic/SED-dependent simulation where drawImage requires a Bandpass;
  the plain achromatic demo1 drawImage route does not apply
- Correlated or non-Gaussian noise is required — GaussianNoise with a single sigma
  only models white pixel noise (noise-whitening/symmetrization are separate procedures)
tools:
- name: GalSim
  role: Provides the GSObject profile classes, galsim.Convolve, drawImage, GaussianNoise,
    and FITS writing used to build and draw the simulated image
  repo: https://github.com/GalSim-developers/GalSim
- name: Python
  role: User-facing API through which the galaxy/PSF objects are constructed, convolved,
    drawn, noised, and written; demo1.py is run as `python demo1.py`
- name: numpy
  role: Underlying numeric types accepted for Image inputs and pixel arrays manipulated
    during drawing and noise addition
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.02.002
    title: 'GalSim: The modular galaxy image simulation toolkit'
  source_content_hash: sha256:d973917c7306ee10f33bc042a239ad71208c3a9e7b08b0df43690fe122ca20f4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/GalSim-developers/GalSim
  tools:
  - GalSim
  - Python
  - numpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# galaxy-psf-convolution-image-drawing

## Summary

Simulate a mock astronomical image by drawing a parametric galaxy profile convolved with a PSF using GalSim's Python API (drawImage), adding Gaussian noise via image.addNoise with a specified sigma, and writing the result to a FITS file. This reproduces the GalSim demo1 tutorial workflow for generating high-fidelity galaxy images.

## When to use

Use when the task is to generate a synthetic galaxy image with a known surface brightness profile (e.g. Gaussian sigma 2 arcsec, flux 1.e5) convolved with a PSF (sigma 1 arcsec), on a 0.2 arcsec/pixel grid, with specified pixel noise.

## When NOT to use

- The galaxy should come from real HST/COSMOS observations rather than a simple parametric model — use GalSim's RealGalaxy with the real_galaxy_catalog (e.g. sample '23.5' or '25.2') instead
- The target is chromatic/SED-dependent simulation where drawImage requires a Bandpass; the plain achromatic demo1 drawImage route does not apply
- Correlated or non-Gaussian noise is required — GaussianNoise with a single sigma only models white pixel noise (noise-whitening/symmetrization are separate procedures)

## Inputs

- Galaxy surface brightness profile parameters (e.g. Gaussian sigma=2 arcsec, flux=1.e5 counts)
- PSF profile parameters (e.g. Gaussian sigma=1 arcsec)
- Pixel scale (0.2 arcsec/pixel) and image size
- Noise model specification (GaussianNoise with sigma=30 counts/pixel)
- Fixed random seed for deterministic reproducibility

## Outputs

- demo1.fits — a FITS image of the PSF-convolved, Gaussian-noised galaxy
- In-memory galsim.Image object carrying the drawn pixel values

## How to apply

Build the galaxy and PSF as GalSim GSObjects using the Python API with demo1 parameter values (galaxy Gaussian sigma 2 arcsec with flux 1.e5 counts; PSF Gaussian sigma 1 arcsec). Form the observed profile with galsim.Convolve([psf, gal]) — convolution with the PSF is required because the observation blurs the intrinsic profile — then draw the image with drawImage on a pixel grid of scale 0.2 arcsec/pixel. Add Gaussian pixel noise by calling image.addNoise(galsim.GaussianNoise(sigma=30)), where sigma=30 is the standard deviation of the counts in each pixel, and supply a fixed random seed baked into the script so the run is fully deterministic. Finally write the drawn, noised image to disk as a FITS file (demo1.fits) in the output directory. The parallel demo1.yaml config-file route with the same seed produces precisely identical images, which is the article's own consistency criterion.

## Related tools

- **GalSim** (Provides the GSObject profile classes, galsim.Convolve, drawImage, GaussianNoise, and FITS writing used to build and draw the simulated image) — https://github.com/GalSim-developers/GalSim
- **Python** (User-facing API through which the galaxy/PSF objects are constructed, convolved, drawn, noised, and written; demo1.py is run as `python demo1.py`)
- **numpy** (Underlying numeric types accepted for Image inputs and pixel arrays manipulated during drawing and noise addition)

## Examples

```
import galsim; img = galsim.Convolve([galsim.Gaussian(sigma=1), galsim.Gaussian(sigma=2, flux=1e5)]).drawImage(scale=0.2); img.addNoise(galsim.GaussianNoise(sigma=30)); img.write('demo1.fits')
```

## Evaluation signals

- demo1.fits exists in the output directory and is a valid FITS image of the expected size and pixel scale (0.2 arcsec/pixel)
- Pixel values precisely match the deposited reference demo1 output because the fixed random seed makes the run fully deterministic
- The observed profile is broader than the intrinsic galaxy (sigma 2 arcsec) due to the PSF (sigma 1 arcsec) convolution — check that the drawn second moment exceeds the input galaxy size
- Pixel-count scatter is consistent with the specified GaussianNoise sigma of 30 counts/pixel
- Running the equivalent demo1.yaml config with the same seed yields precisely identical images to the Python-script route

## Limitations

- Demo scripts are designed to be run from the examples/ directory (`python demo1.py`); some access files in subdirectories, so they fail from other locations
- Deterministic, byte-identical reproduction requires the fixed random seed baked into the script; changing the seed changes the noise realization
- The default FFT method for drawing chromatic objects is fairly slow because it integrates the image over the bandpass (not an issue for the achromatic demo1 case, but limits extension of this workflow)
- The bulk of GalSim's calculations are carried out in C++, so performance depends on the compiled installation (FFTW is a non-pip-installable dependency)

## Evidence

- [results] Draw an image of a galaxy convolved with a PSF and write it to disk: [evidence span withheld — anchor status: not_found]
- [results] Add Gaussian noise to the image: "image.addNoise(galsim.GaussianNoise(sigma=noise))"
- [intro] GalSim purpose and interface: "GalSim is open-source software for simulating images of astronomical objects (stars, galaxies) in a variety of ways."
- [readme] Demo scripts location and invocation: [evidence span withheld — anchor status: external_doc]
- [results] Script/config equivalence under a fixed seed: "This is the same seed that is used in demo2.yaml, which means the images produced by the two methods will be precisely identical."
