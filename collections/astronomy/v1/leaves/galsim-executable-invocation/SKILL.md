---
name: galsim-executable-invocation
description: Use when a simulation task must run purely from a config file without
  Python scripting — e.g. reproducing demo*.yaml images of a Gaussian galaxy convolved
  with a Gaussian PSF with Gaussian noise.
when_to_use_negative:
- When custom simulation logic is needed beyond registered config builders (e.g. arbitrary
  profiles or custom builders) — a Python script importing galsim is the appropriate
  route instead
- When the user already has a working demo*.py script; the config route is for those
  who prefer not to work in Python and produces identical output, so switching adds
  no benefit
- When the required builder type is not registered in the config parser (custom image,
  noise, WCS, bandpass, or sensor builders need a registered type name in an importable
  module loaded via the top-level modules field)
edam_operation: http://edamontology.org/operation_2426
tools:
- name: galsim executable
  role: Parses the YAML config dictionary and drives the simulation end-to-end, including
    image construction and FITS output
  repo: https://github.com/GalSim-developers/GalSim
- name: GalSim
  role: Underlying simulation toolkit whose bulk calculations are in C++ with a Python
    interface, invoked here via config files
  repo: https://github.com/GalSim-developers/GalSim
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ascom.2015.02.002
    title: 'GalSim: The modular galaxy image simulation toolkit'
  source_content_hash: sha256:69159f1e17e69092a5fd52d97ae3dedb3bbda79f7e6b660d383b391753a9781a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/GalSim-developers/GalSim
  tools:
  - GalSim
  - galsim executable
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# galsim-executable-invocation

## Summary

Run GalSim galaxy image simulations without writing Python code by invoking the `galsim` executable directly on YAML config files (e.g. demo1.yaml), which parse a dictionary of galaxy, PSF, image, and noise settings and write the simulated image to disk.

## When to use

Use when a simulation task must run purely from a config file without Python scripting — e.g. reproducing demo*.yaml images of a Gaussian galaxy convolved with a Gaussian PSF with Gaussian noise.

## When NOT to use

- When custom simulation logic is needed beyond registered config builders (e.g. arbitrary profiles or custom builders) — a Python script importing galsim is the appropriate route instead
- When the user already has a working demo*.py script; the config route is for those who prefer not to work in Python and produces identical output, so switching adds no benefit
- When the required builder type is not registered in the config parser (custom image, noise, WCS, bandpass, or sensor builders need a registered type name in an importable module loaded via the top-level modules field)

## Inputs

- YAML config file (e.g. demo1.yaml) defining galaxy profile, PSF, image, noise, and output fields
- Optional command-line parameter overrides appended after the config file name
- Optional -n and -j command-line options for splitting jobs across machines

## Outputs

- FITS image file written to the directory/file_name specified in the config's output field (e.g. output_yaml/demo1.fits)

## How to apply

Locate the YAML config file (e.g. examples/demo1.yaml in the GalSim repository), which defines a Gaussian galaxy (sigma=2 arcsec, flux=1.e5), a Gaussian PSF (sigma=1 arcsec, flux defaulting to 1), and image settings (pixel_scale=0.2 arcsec/pixel, Gaussian noise sigma=30 ADU). Invoke the executable as `galsim demo1.yaml`; the parser builds the image using the default image type 'Single' and output type 'Fits', writing to the directory and file_name given in the config's output field (e.g. dir: output_yaml, file_name: demo1.fits). Additional or changed parameters can be supplied on the command line after the config file name, and large jobs can be split across machines with the -n and -j options. Because demo*.yaml files are a parallel sequence to the demo*.py scripts (same random seed), the image produced via config file should be precisely identical to the scripted equivalent, which is the strongest correctness check.

## Related tools

- **galsim executable** (Parses the YAML config dictionary and drives the simulation end-to-end, including image construction and FITS output) — https://github.com/GalSim-developers/GalSim
- **GalSim** (Underlying simulation toolkit whose bulk calculations are in C++ with a Python interface, invoked here via config files) — https://github.com/GalSim-developers/GalSim

## Examples

```
galsim demo1.yaml
```

## Evaluation signals

- The output FITS file (e.g. output_yaml/demo1.fits) exists and is readable (file_exists / file_format_is checks)
- Image dimensions and pixel scale match the config (pixel_scale=0.2 arcsec/pixel for demo1)
- With the same random seed as the equivalent demo*.py script, the config-file image is precisely identical (e.g. demo2.yaml vs demo2.py)
- Flux/noise properties are consistent: galaxy flux=1.e5, PSF flux defaulting to 1, Gaussian noise sigma=30 ADU as specified in the config

## Limitations

- The default FFT method for drawing chromatic objects is fairly slow because it integrates the image over the bandpass
- Custom builder types (image, noise, WCS, bandpass, sensor) require registering the function with a type name in an importable Python module and loading it via the top-level modules field
- Demo scripts/configs are designed to be run from the examples/ directory; some access files in subdirectories and will not work correctly from other locations

## Evidence

- [methods] Normal config invocation: "The normal way to run a GalSim simulation using a config file is ``galsim config.yaml``"
- [methods] YAML read by executable: "Our example config files are all yaml files, which are read using the executable ``galsim``."
- [methods] Command-line overrides: "you can provide the changed (or new) parameters on the command line after the name of the config file"
- [methods] Job splitting options: "We make this splitting very easy using the command line options ``-n`` and ``-j``."
- [results] Identical outputs via seed: "This is the same seed that is used in demo2.yaml, which means the images produced by the two methods will be precisely identical."
- [readme] Config files for non-Python users: "the code can operate directly on "config" files, for those users who prefer not to work in Python"
- [readme] Parallel demo yaml sequence: [evidence span withheld — anchor status: external_doc]
