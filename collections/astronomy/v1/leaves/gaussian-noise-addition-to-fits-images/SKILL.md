---
name: gaussian-noise-addition-to-fits-images
description: Use when simulating an astronomical image that needs realistic pixel-level
  noise after drawing a galaxy convolved with a PSF, e.g. reproducing demo1-style
  GalSim outputs with a fixed random seed.
when_to_use_negative:
- The image already contains noise, e.g. RealGalaxy images from the COSMOS catalog,
  where a noise-whitening procedure rather than fresh Gaussian noise is required.
- Correlated noise is needed, where whitening or symmetrizing an existing noise field
  is the correct operation instead of adding white Gaussian noise.
- The simulation is driven by a config file run through the galsim executable and
  noise should be specified declaratively rather than via the Python API.
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: GalSim
  role: Provides the GaussianNoise class and Image.addNoise method used to add per-pixel
    Gaussian noise to the drawn image
  repo: https://github.com/GalSim-developers/GalSim
- name: Python
  role: User interface language in which the demo1.py script builds profiles, draws
    the image, and calls addNoise
- name: numpy
  role: Numeric backend for the Image arrays that GalSim operates on when adding noise
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ascom.2015.02.002
    title: 'GalSim: The modular galaxy image simulation toolkit'
  source_content_hash: sha256:86f8f5dfe0c6336fab3f167736c52295849e082ed27da8031f66b4620d494e1c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/GalSim-developers/GalSim
  tools:
  - GalSim
  - Python
  - numpy
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# gaussian-noise-addition-to-fits-images

## Summary

Adds Gaussian (white) noise to a GalSim-drawn galaxy image using image.addNoise with a galsim.GaussianNoise object at a specified sigma, then writes the result to a FITS file, replicating the deterministic demo1.py workflow.

## When to use

Use when simulating an astronomical image that needs realistic pixel-level noise after drawing a galaxy convolved with a PSF, e.g. reproducing demo1-style GalSim outputs with a fixed random seed.

## When NOT to use

- The image already contains noise, e.g. RealGalaxy images from the COSMOS catalog, where a noise-whitening procedure rather than fresh Gaussian noise is required.
- Correlated noise is needed, where whitening or symmetrizing an existing noise field is the correct operation instead of adding white Gaussian noise.
- The simulation is driven by a config file run through the galsim executable and noise should be specified declaratively rather than via the Python API.

## Inputs

- GalSim-drawn image of a galaxy convolved with a PSF (drawImage output)
- Noise standard deviation sigma in counts per pixel (demo1 uses sigma=30)
- Fixed random seed for deterministic noise realization

## Outputs

- Noisy image with per-pixel Gaussian noise (sigma=30 counts)
- FITS file on disk (demo1.fits) in the output directory

## How to apply

Build the galaxy and PSF surface brightness profiles with the GalSim Python API using the demo1 parameter values (galaxy sigma 2 arcsec, flux 1.e5 counts; PSF sigma 1 arcsec; pixel_scale 0.2 arcsec/pixel) and draw the convolved image with drawImage. Then call image.addNoise(galsim.GaussianNoise(sigma=noise)) with sigma 30, the standard deviation of the counts in each pixel, using the fixed random seed baked into the script so the run is fully deterministic. Write the resulting image to disk in FITS format as demo1.fits in the output directory. The rationale is that GaussianNoise applies uncorrelated per-pixel noise, the standard observational noise model in GalSim simulations.

## Related tools

- **GalSim** (Provides the GaussianNoise class and Image.addNoise method used to add per-pixel Gaussian noise to the drawn image) — https://github.com/GalSim-developers/GalSim
- **Python** (User interface language in which the demo1.py script builds profiles, draws the image, and calls addNoise)
- **numpy** (Numeric backend for the Image arrays that GalSim operates on when adding noise)

## Examples

```
image.addNoise(galsim.GaussianNoise(sigma=30))
```

## Evaluation signals

- demo1.fits exists in the output directory and is a valid FITS image of the expected size
- Image pixel values precisely match the deposited reference demo output, since the fixed random seed makes the run fully deterministic
- Per-pixel noise scatter is consistent with the specified sigma=30 counts standard deviation
- Re-running the script reproduces bit-identical images (deterministic seeded noise)

## Limitations

- GaussianNoise applies uncorrelated (white) noise; correlated noise requires a separate whitening step on the image after it is drawn.
- Determinism depends on the fixed random seed; changing the seed changes the noise realization and breaks exact pixel-level comparison against reference outputs.
- Demo scripts are designed to be run from the examples/ directory and may fail from other locations.

## Evidence

- [results] Add Gaussian noise to the image: [evidence span withheld — anchor status: not_found]
