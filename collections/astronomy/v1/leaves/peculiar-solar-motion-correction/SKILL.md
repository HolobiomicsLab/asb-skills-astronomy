---
name: peculiar-solar-motion-correction
description: Use when kinematic data sets compiled under different Galactic parameters
  must be converted to a chosen parameter frame, and terminal velocities need the
  small peculiar-solar-motion correction applied consistently.
when_to_use_negative:
- You need the original published values as in each source reference — galkin/data/data_*.dat
  and output/rawdata.dat are not normalised to the selected input parameters
- You are working outside the Galactocentric-radius range the compilation is optimised
  for (3–20 kpc)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0621
tools:
- name: galkin
  role: Python package providing the data-handling routines that convert each kinematic
    data set to the chosen Galactic parameters, including the peculiar-solar-motion
    correction of terminal velocities
  repo: github.com/galkintool/galkin
- name: galkin_data.py
  role: Script that reads the input parameter file and drives the conversion, outputting
    vcdata.dat, posdata.dat, and rawdata.dat
  repo: github.com/galkintool/galkin
- name: numpy
  role: Required Python dependency for running the galkin conversion routines
- name: galkin_data_fast.py
  role: In-memory variant of the data retriever for repeated on-the-fly parameter
    scanning without file input/output
  repo: github.com/galkintool/galkin
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.softx.2016.12.006
    title: 'galkin: A new compilation of Milky Way rotation curve data'
  source_content_hash: sha256:8dd9ffc6c51293c3c6c9226ad40bd684fa7ab91a22b33a1eadf9f74f5cb0150c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - galkin
  - galkin_data.py
  - galkin_data_fast.py
  - numpy
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# peculiar-solar-motion-correction

## Summary

Applies the peculiar-solar-motion correction to terminal-velocity kinematic data when re-normalising heterogeneous Milky Way rotation-curve measurements to a user-chosen Galactic parameter set (e.g. R0=7.5 or 8.5 kpc) using galkin.

## When to use

Use when kinematic data sets compiled under different Galactic parameters must be converted to a chosen parameter frame, and terminal velocities need the small peculiar-solar-motion correction applied consistently.

## When NOT to use

- You need the original published values as in each source reference — galkin/data/data_*.dat and output/rawdata.dat are not normalised to the selected input parameters
- You are working outside the Galactocentric-radius range the compilation is optimised for (3–20 kpc)

## Inputs

- input parameter file (e.g. inputpars.txt) specifying the chosen Galactic parameter set (R0, peculiar solar motion)
- original kinematic data files under galkin/data/data_*.dat retrieved from the source references in Table 1 of the manuscript

## Outputs

- output/vcdata.dat — rotation-curve velocity data normalised to the chosen parameters
- output/posdata.dat — positional data normalised to the chosen parameters
- output/rawdata.dat — raw data as in the original references (not normalised)

## How to apply

Install the galkin package with 'python setup.py install' under Python 2.7.6+ (numpy required), then prepare an input parameter file specifying the chosen Galactic parameters — e.g. R0=7.5 kpc plus the assumed peculiar solar motion — and run 'python galkin_data.py inputpars.txt'. galkin reprocesses the original published data under galkin/data/data_*.dat (2780 gas, star, and maser measurements from Table 1 references), converting each data set consistently to the chosen parameter frame; the peculiar-solar-motion correction to terminal velocities is small and depends on the assumed solar motion. Validate by diffing the regenerated output/vcdata.dat and output/posdata.dat against the shipped sample files (posdata_R75.dat/vcdata_R75.dat for R0=7.5, posdata_R85.dat/vcdata_R85.dat for R0=8.5, and output/vcdata_baseline.dat/posdata_baseline.dat for the default setup). Do not use the raw output/rawdata.dat or galkin/data/data_*.dat files directly, since they are given as in the original references and are not normalised to the selected input parameters.

## Related tools

- **galkin** (Python package providing the data-handling routines that convert each kinematic data set to the chosen Galactic parameters, including the peculiar-solar-motion correction of terminal velocities) — github.com/galkintool/galkin
- **galkin_data.py** (Script that reads the input parameter file and drives the conversion, outputting vcdata.dat, posdata.dat, and rawdata.dat) — github.com/galkintool/galkin
- **numpy** (Required Python dependency for running the galkin conversion routines)
- **galkin_data_fast.py** (In-memory variant of the data retriever for repeated on-the-fly parameter scanning without file input/output) — github.com/galkintool/galkin

## Examples

```
python galkin_data.py inputpars.txt
```

## Evaluation signals

- Regenerated output/vcdata.dat and output/posdata.dat for R0=7.5 kpc match the shipped sample files posdata_R75.dat and vcdata_R75.dat
- Regenerated outputs for R0=8.5 kpc match the shipped posdata_R85.dat and vcdata_R85.dat
- Default parameter run matches output/vcdata_baseline.dat and output/posdata_baseline.dat for checking purposes
- The rawdata output is unchanged from the original references (unnormalised), confirming only vcdata/posdata carry the conversion
- The peculiar-solar-motion correction on terminal velocities is small and consistent with the assumed solar motion value

## Limitations

- The peculiar-solar-motion correction to terminal velocities is small but depends on the assumed solar motion, so results vary with that choice
- Raw data files (galkin/data/data_*.dat and output/rawdata.dat) are given as in the original references and must not be used directly unless fully aware of each reference's assumptions and nomenclature
- Code runs under Python 2.7.6 (or higher); wx is required only for the user-friendly window mode and astropy only for optional coordinate conversion
- Source references use different Galactocentric parameters and definitions and often overlap in observed sources, which the unified conversion is designed to handle

## Evidence

- [methods] galkin processes the original data sets of the selected references and converts each data set consistently for the chosen values of the Galactic parameters: [evidence span withheld — anchor status: not_found]
- [results] the peculiar-solar-motion correction applied to terminal velocities being small and dependent on the assumed solar motion: [evidence span withheld — anchor status: not_found]
