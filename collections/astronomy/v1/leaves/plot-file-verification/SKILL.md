---
name: plot-file-verification
description: Use when you need to visualise and sanity-check the output of galkin_data.py
  (output/vcdata.dat and output/posdata.dat) by rendering plots and confirming them
  against the baseline reference files.
when_to_use_negative:
- You need programmatic access to the rotation-curve values themselves rather than
  rendered plots — use galkin_data_fast.py for in-memory retrieval instead
- You intend to work with the original kinematic data in galkin/data/data_*.dat or
  output/rawdata.dat, which are not normalised to the selected input parameters and
  are explicitly flagged as unsafe to use directly unless all assumptions of each
  original reference are understood
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: galkin_plotter.py
  role: Ready-to-use script that plots the data outputted by galkin_data.py
  repo: github.com/galkintool/galkin
- name: galkin
  role: Python package providing the data-handling routines and plotting backend consumed
    by the plotter
  repo: github.com/galkintool/galkin
- name: matplotlib
  role: Python package required for the plotting backend
- name: numpy
  role: Required python package dependency of galkin
- name: galkin_data.py
  role: Upstream script that produces output/vcdata.dat and output/posdata.dat fed
    to the plotter
  repo: github.com/galkintool/galkin
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.softx.2016.12.006
    title: 'galkin: A new compilation of Milky Way rotation curve data'
  source_content_hash: sha256:5cc145c9aa117c1cc16c842043bfa3069b1c35d5630617ee144c129183aff630
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - galkin
  - galkin_data.py
  - galkin_plotter.py
  - matplotlib
  - numpy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# plot-file-verification

## Summary

Generates and validates the demonstrative plots (tracer spatial distribution and Milky Way rotation curve) produced by galkin_plotter.py from the galkin_data.py output files, verifying that each rendered figure exists, is readable, and matches the baseline reference data.

## When to use

Use when you need to visualise and sanity-check the output of galkin_data.py (output/vcdata.dat and output/posdata.dat) by rendering plots and confirming them against the baseline reference files.

## When NOT to use

- You need programmatic access to the rotation-curve values themselves rather than rendered plots — use galkin_data_fast.py for in-memory retrieval instead
- You intend to work with the original kinematic data in galkin/data/data_*.dat or output/rawdata.dat, which are not normalised to the selected input parameters and are explicitly flagged as unsafe to use directly unless all assumptions of each original reference are understood

## Inputs

- output/vcdata.dat — rotation curve data produced by galkin_data.py for the chosen Galactic parameters
- output/posdata.dat — positional (tracer spatial distribution) data produced by galkin_data.py
- output/vcdata_baseline.dat and output/posdata_baseline.dat — baseline reference files for checking purposes

## Outputs

- Rendered figure of the spatial distribution of the kinematic tracers (gas, stars, masers)
- Rendered figure of the Milky Way rotation curve optimised for Galactocentric radii between 3 and 20 kpc
- Verified image files (exist, readable, plotted data consistent with baseline)

## How to apply

First obtain the baseline output files output/vcdata.dat and output/posdata.dat that galkin_data.py produces for the default Galactic parameter setup (optimised for Galactocentric radii between 3 and 20 kpc, covering gas, stars and masers). Then run the ready-to-use plotter with 'python galkin_plotter.py output/vcdata.dat output/posdata.dat', which uses the matplotlib plotting backend of the galkin package. The plotter renders two figure classes: the spatial distribution of the kinematic tracers and the inferred rotation curve. Validate that each figure file exists, is a readable image, and that the plotted data points/curve are consistent with the baseline checking files output/vcdata_baseline.dat and output/posdata_baseline.dat distributed in the repository.

## Related tools

- **galkin_plotter.py** (Ready-to-use script that plots the data outputted by galkin_data.py) — github.com/galkintool/galkin
- **galkin** (Python package providing the data-handling routines and plotting backend consumed by the plotter) — github.com/galkintool/galkin
- **matplotlib** (Python package required for the plotting backend)
- **numpy** (Required python package dependency of galkin)
- **galkin_data.py** (Upstream script that produces output/vcdata.dat and output/posdata.dat fed to the plotter) — github.com/galkintool/galkin

## Examples

```
python galkin_plotter.py output/vcdata.dat output/posdata.dat
```

## Evaluation signals

- Each output figure file exists and is a readable image
- Plotted tracer spatial distribution matches the points in output/posdata_baseline.dat
- Plotted rotation curve matches output/vcdata_baseline.dat for the default (baseline) Galactic parameter setup
- The plotted curve spans the intended Galactocentric radii range (3–20 kpc) and includes gas, stars and maser tracers
- Running the plotter against alternate parameter outputs (e.g. R0=8.5 or 7.5 kpc sample files) produces correspondingly shifted curves rather than baseline values

## Limitations

- The plotter handles output data of galkin_data.py; it does not validate the underlying physics — consistency checks rely on comparison to the baseline files
- The raw data (output/rawdata.dat) are given as in the original reference and not normalised to the selected input parameters, so plots built from them directly are only meaningful if all assumptions and nomenclature of each original reference are understood
- Requires the wx package only in 'User-friendly window' mode of galkin_data.py; astropy is needed only if coordinate conversion is done with astropy

## Evidence
