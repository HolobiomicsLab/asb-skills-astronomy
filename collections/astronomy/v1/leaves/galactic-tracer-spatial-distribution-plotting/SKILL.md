---
name: galactic-tracer-spatial-distribution-plotting
description: Use when you have output/vcdata.dat and output/posdata.dat produced by
  galkin_data.py and need to visualise the tracer spatial distribution and the inferred
  rotation curve of the Galaxy.
when_to_use_negative:
- You need data retrieval or parameter scanning rather than plotting — use galkin_data.py
  or galkin_data_fast.py instead.
- You intend to use the raw original data in output/rawdata.dat directly for plotting
  — the README warns these data are not normalised to the selected input parameters
  and should only be used if fully aware of each reference's assumptions.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: galkin_plotter.py
  role: Ready-to-use plotter script invoked on the vcdata/posdata output files to
    produce the demonstrative figures
  repo: github.com/galkintool/galkin
- name: galkin
  role: Python package providing the plotting backend (galkin/plots.py) used by galkin_plotter.py
  repo: github.com/galkintool/galkin
- name: matplotlib
  role: Required python package underpinning the plotting backend of the galkin package
- name: numpy
  role: Required python package for the galkin code used during plotting
- name: galkin_data.py
  role: Upstream script that generates the vcdata.dat and posdata.dat files the plotter
    consumes
  repo: github.com/galkintool/galkin
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.softx.2016.12.006
    title: 'galkin: A new compilation of Milky Way rotation curve data'
  source_content_hash: sha256:21c92f9346cb80ef4d5cf9c793e76941baec57ca959ee61b75988d0d30ac77fe
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
yes_rate: 0.4
n_claims: 15
---

# galactic-tracer-spatial-distribution-plotting

## Summary

Render demonstrative figures of the Milky Way rotation curve and the spatial distribution of kinematic tracers (gas, stars, masers) from galkin_data.py outputs using the galkin package's ready-to-use plotter, galkin_plotter.py, built on the matplotlib backend.

## When to use

Use when you have output/vcdata.dat and output/posdata.dat produced by galkin_data.py and need to visualise the tracer spatial distribution and the inferred rotation curve of the Galaxy.

## When NOT to use

- You need data retrieval or parameter scanning rather than plotting — use galkin_data.py or galkin_data_fast.py instead.
- You intend to use the raw original data in output/rawdata.dat directly for plotting — the README warns these data are not normalised to the selected input parameters and should only be used if fully aware of each reference's assumptions.

## Inputs

- output/vcdata.dat — rotation curve data outputted by galkin_data.py
- output/posdata.dat — positional data of the kinematic tracers outputted by galkin_data.py
- output/vcdata_baseline.dat and output/posdata_baseline.dat — baseline reference files for checking

## Outputs

- Rendered figure files showing the spatial distribution of the kinematic tracers (gas, stars, masers)
- Rendered figure files showing the rotation curve of the Milky Way optimised for Galactocentric radii between 3 and 20 kpc

## How to apply

First run galkin_data.py (e.g. 'python galkin_data.py inputpars.txt' with the default Galactic parameter setup) so it writes output/vcdata.dat (rotation curve) and output/posdata.dat (tracer positions). Then invoke the ready-to-use plotter with 'python galkin_plotter.py output/vcdata.dat output/posdata.dat', which relies on the galkin package's plotting routines (galkin/plots.py) and requires matplotlib and numpy. The rendered figures cover both the spatial distribution of the tracers and the rotation curve, which is optimised for Galactocentric radii between 3 and 20 kpc. Save the produced figure files and verify each one exists and is a readable image. For validation, compare the plotted data against the baseline reference files output/vcdata_baseline.dat and output/posdata_baseline.dat shipped for checking purposes.

## Related tools

- **galkin_plotter.py** (Ready-to-use plotter script invoked on the vcdata/posdata output files to produce the demonstrative figures) — github.com/galkintool/galkin
- **galkin** (Python package providing the plotting backend (galkin/plots.py) used by galkin_plotter.py) — github.com/galkintool/galkin
- **matplotlib** (Required python package underpinning the plotting backend of the galkin package)
- **numpy** (Required python package for the galkin code used during plotting)
- **galkin_data.py** (Upstream script that generates the vcdata.dat and posdata.dat files the plotter consumes) — github.com/galkintool/galkin

## Examples

```
python galkin_plotter.py output/vcdata.dat output/posdata.dat
```

## Evaluation signals

- Each rendered figure file exists and is a readable image.
- Plotted data points/curve are consistent with the baseline reference files output/vcdata_baseline.dat and output/posdata_baseline.dat.
- The rotation-curve figure covers Galactocentric radii between 3 and 20 kpc, matching the compilation's optimised range.
- The tracer spatial-distribution figure reflects the 2780 gas, star and maser measurements in the galkin compilation.

## Limitations

- The plotter handles the output of galkin_data.py; the raw data in output/rawdata.dat are given as in the original references and not normalised to the selected input parameters, so plotting them directly is discouraged unless the user is fully aware of each reference's assumptions and nomenclature.
- The plotting backend requires matplotlib and numpy (and wx only in 'User-friendly window' mode of galkin_data.py).
- The script runs under python 2.7.6 or higher as distributed in galkin version 1.0.

## Evidence
