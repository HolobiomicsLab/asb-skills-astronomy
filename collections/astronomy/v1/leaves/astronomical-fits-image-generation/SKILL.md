---
name: astronomical-fits-image-generation
description: Use when a simulation image must be produced without writing Python code
  — by feeding a YAML config such as demo1.yaml to the galsim executable and writing
  the rendered galaxy+PSF image to a FITS file.
when_to_use_negative:
- You need interactive, programmatic control inside a Python session (e.g. forming
  ChromaticObjects by multiplying a GSObject by an SED, or drawImage with a Bandpass)
  — use the Python API (demo*.py) instead of the config route.
- The goal is to measure or fit existing images (e.g. shear inference / metacalibration)
  rather than simulate new ones — this skill only renders and writes simulated FITS
  images.
- A custom image, noise, WCS, bandpass, or sensor builder is needed but has not been
  registered with a type name in an importable module loaded via the top-level modules
  field — the config parser cannot resolve unknown types on its own.
tools:
- name: galsim executable
  role: Parses the YAML config dictionary and drives image generation plus FITS output
    without Python code
  repo: https://github.com/GalSim-developers/GalSim
- name: GalSim
  role: Provides the config schema (gal, psf, image, output fields), default image
    type 'Single' and output type 'Fits', and the -n/-j job-splitting options
  repo: https://github.com/GalSim-developers/GalSim
- name: Python
  role: Alternative API path mirrored by the config files (demo*.py vs demo*.yaml);
    not required for the config-driven route
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.02.002
    title: 'GalSim: The modular galaxy image simulation toolkit'
  source_content_hash: sha256:c3b2cdb0006fa646d494794a01705c9acbcfd699aa4992c15053c2d0529493d1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/GalSim-developers/GalSim
  tools:
  - GalSim
  - Python
  - galsim executable
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# astronomical-fits-image-generation

## Summary

Config-driven generation of simulated astronomical images: a GalSim YAML config (e.g. demo1.yaml, defining a Gaussian galaxy with sigma=2 arcsec and flux=1.e5 convolved with a Gaussian PSF of sigma=1 arcsec) is parsed by the galsim executable, which draws the image and writes it to disk as a FITS file with no Python code required.

## When to use

Use when a simulation image must be produced without writing Python code — by feeding a YAML config such as demo1.yaml to the galsim executable and writing the rendered galaxy+PSF image to a FITS file.

## When NOT to use

- You need interactive, programmatic control inside a Python session (e.g. forming ChromaticObjects by multiplying a GSObject by an SED, or drawImage with a Bandpass) — use the Python API (demo*.py) instead of the config route.
- The goal is to measure or fit existing images (e.g. shear inference / metacalibration) rather than simulate new ones — this skill only renders and writes simulated FITS images.
- A custom image, noise, WCS, bandpass, or sensor builder is needed but has not been registered with a type name in an importable module loaded via the top-level modules field — the config parser cannot resolve unknown types on its own.

## Inputs

- YAML config file with gal, psf, image, and output fields — e.g. examples/demo1.yaml: Gaussian galaxy (sigma=2 arcsec, flux=1.e5), Gaussian PSF (sigma=1 arcsec), pixel_scale=0.2 arcsec/pixel, Gaussian noise sigma=30 ADU
- galsim executable (bin/galsim), installed via `pip install galsim` or conda-forge
- Optional command-line overrides (changed or new parameters) appended after the config file name

## Outputs

- FITS image file at the path given by the config's output field (dir: output_yaml, file_name: demo1.fits)
- Rendered single-galaxy image (galaxy convolved with PSF, Gaussian noise added) at pixel_scale=0.2 arcsec/pixel

## How to apply

Locate or author a YAML config such as examples/demo1.yaml in the GalSim repository (github.com/GalSim-developers/GalSim), where the gal field defines a Gaussian galaxy (sigma=2 arcsec, flux=1.e5), the psf field a Gaussian PSF (sigma=1 arcsec, flux defaulting to 1), and the image field pixel_scale=0.2 arcsec/pixel with Gaussian noise sigma=30 ADU. Invoke the executable (`galsim demo1.yaml`); it parses the YAML dictionary and builds the image using the default image type 'Single' and output type 'Fits', writing to the dir and file_name given in the output field (output_yaml/demo1.fits). Changed or new parameters can be supplied on the command line after the config file name, and long simulation jobs can be split across machines with the -n and -j options. The rationale is exact parity with the Python route: using the same random seed as demo2.yaml, the script and config-file methods produce precisely identical images, so config-only operation is a faithful substitute for writing Python code.

## Related tools

- **galsim executable** (Parses the YAML config dictionary and drives image generation plus FITS output without Python code) — https://github.com/GalSim-developers/GalSim
- **GalSim** (Provides the config schema (gal, psf, image, output fields), default image type 'Single' and output type 'Fits', and the -n/-j job-splitting options) — https://github.com/GalSim-developers/GalSim
- **Python** (Alternative API path mirrored by the config files (demo*.py vs demo*.yaml); not required for the config-driven route)

## Examples

```
galsim demo1.yaml
```

## Evaluation signals

- output_yaml/demo1.fits exists and passes file_exists and file_format_is checks as a readable FITS image
- Output path matches the config's output field (dir: output_yaml, file_name: demo1.fits)
- A config run using the same random seed as demo2.yaml produces images precisely identical to the equivalent demo*.py script output
- Image content matches the config: Gaussian galaxy (sigma=2 arcsec, flux=1.e5) convolved with Gaussian PSF (sigma=1 arcsec) at pixel_scale=0.2 arcsec/pixel with Gaussian noise sigma=30 ADU
- Omitted type fields resolve to documented defaults: image type 'Single' and output type 'Fits'

## Limitations

- All demo scripts and configs are designed to be run in the examples/ directory and some access files in its subdirectories, so they would not work correctly from other locations.
- Capabilities beyond the built-in config vocabulary require registering custom builders with a type name in an importable module and loading it via the top-level modules field before processing.
- The default FFT method for drawing chromatic objects is fairly slow because it needs to integrate the image over the bandpass.

## Evidence

- [methods] Config-driven simulations are run by parsing YAML with the galsim executable: "The normal way to run a GalSim simulation using a config file is ``galsim config.yaml``"
- [methods] GalSim example configs are YAML files read by the galsim executable: "Our example config files are all yaml files, which are read using the executable ``galsim``."
- [results] The pipeline draws a galaxy convolved with a PSF and writes it to disk: "We draw an image of a single galaxy convolved with a PSF and write it to disk."
- [results] Same seed makes script and config outputs precisely identical: [evidence span withheld — anchor status: not_found]
- [readme] GalSim can operate on config files for users who prefer not to work in Python: "In addition, the code can operate directly on "config" files, for those users who prefer not to work in Python."
