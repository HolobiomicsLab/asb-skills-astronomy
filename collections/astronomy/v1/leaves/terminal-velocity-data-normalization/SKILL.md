---
name: terminal-velocity-data-normalization
description: Use when kinematic measurements (e.g. terminal velocities from gas tracers)
  were published under different Galactocentric parameter assumptions (R0, solar motion)
  and must be made consistent before combining into a unified rotation curve.
when_to_use_negative:
- The user wants the raw published values as given in each original reference — galkin/data/data_*.dat
  and output/rawdata.dat are not normalized to the selected input parameters and should
  only be used with full awareness of each reference's assumptions and nomenclature
- Data are already expressed in the target Galactic parameter frame — reprocessing
  through galkin_data.py would add no value
- The analysis requires Galactocentric radii outside ~3–20 kpc, the range the galkin
  compilation is optimised for
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: galkin
  role: Python package providing the data handling routines (processdata.py, functions.py)
    that import and convert the original kinematic data to the chosen parameter set
  repo: github.com/galkintool/galkin
- name: galkin_data.py
  role: Script that retrieves and outputs the data given input options and parameters
    from a window or input parameter file
  repo: github.com/galkintool/galkin
- name: galkin_data_fast.py
  role: In-memory variant for repeatedly applying the conversion on the fly, e.g.
    scanning over input parameters
  repo: github.com/galkintool/galkin
- name: Python 2.7.6
  role: Runtime environment for galkin (or higher)
  repo: github.com/galkintool/galkin
- name: numpy
  role: Required dependency for galkin's data processing
  repo: github.com/galkintool/galkin
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.softx.2016.12.006
    title: 'galkin: A new compilation of Milky Way rotation curve data'
  source_content_hash: sha256:12cd8e3b006f6b5ac35521d3ec974addfb033bd196e9fa760cdd407455604659
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python 2.7.6
  - galkin
  - galkin_data.py
  - galkin_data_fast.py
  - numpy
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# terminal-velocity-data-normalization

## Summary

Normalize Milky Way rotation-curve tracer data (gas, stars, masers, including terminal velocities) compiled under heterogeneous Galactic parameters onto a single user-chosen parameter frame (e.g. R0, peculiar solar motion) using the galkin package.

## When to use

Use when kinematic measurements (e.g. terminal velocities from gas tracers) were published under different Galactocentric parameter assumptions (R0, solar motion) and must be made consistent before combining into a unified rotation curve.

## When NOT to use

- The user wants the raw published values as given in each original reference — galkin/data/data_*.dat and output/rawdata.dat are not normalized to the selected input parameters and should only be used with full awareness of each reference's assumptions and nomenclature
- Data are already expressed in the target Galactic parameter frame — reprocessing through galkin_data.py would add no value
- The analysis requires Galactocentric radii outside ~3–20 kpc, the range the galkin compilation is optimised for

## Inputs

- galkin/data/data_*.dat — original kinematic data retrieved from the source references in Table 1 of the manuscript
- Input parameter file (e.g. inputpars.txt) specifying target Galactic parameters (R0, peculiar solar motion)
- Terminal-velocity and other tracer measurements (gas, stars, masers) from the original references

## Outputs

- output/vcdata.dat — rotation curve data normalized to the chosen parameters
- output/posdata.dat — positional data normalized to the chosen parameters
- output/rawdata.dat — raw data in the original references (not normalized)

## How to apply

Install the galkin package with 'python setup.py install' under Python 2.7.6+ with numpy (matplotlib, wx, astropy as needed). Do NOT read galkin/data/data_*.dat directly — those hold the original published values from each source reference and are not normalized; instead let the package reprocess them. Write an input parameter file specifying the target parameter set (e.g. R0=7.5 or 8.5 kpc plus the chosen peculiar solar motion) and run 'python galkin_data.py inputpars.txt', which imports and processes the original kinematic data (gas, stars, masers, 2780 measurements over ~3–20 kpc Galactocentric radii) via the galkin/processdata.py routines. The peculiar-solar-motion correction applied to terminal velocities is small and depends on the assumed solar motion, so verify sensitivity by rerunning with an alternative R0. Validate each regenerated output/vcdata.dat and output/posdata.dat by diffing against the shipped sample files posdata_R75.dat/vcdata_R75.dat (R0=7.5) or posdata_R85.dat/vcdata_R85.dat (R0=8.5), or against output/vcdata_baseline.dat and output/posdata_baseline.dat for the default setup. For repeated parameter scans, use galkin_data_fast.py to process data in-memory without file I/O.

## Related tools

- **galkin** (Python package providing the data handling routines (processdata.py, functions.py) that import and convert the original kinematic data to the chosen parameter set) — github.com/galkintool/galkin
- **galkin_data.py** (Script that retrieves and outputs the data given input options and parameters from a window or input parameter file) — github.com/galkintool/galkin
- **galkin_data_fast.py** (In-memory variant for repeatedly applying the conversion on the fly, e.g. scanning over input parameters) — github.com/galkintool/galkin
- **Python 2.7.6** (Runtime environment for galkin (or higher)) — github.com/galkintool/galkin
- **numpy** (Required dependency for galkin's data processing) — github.com/galkintool/galkin

## Examples

```
python galkin_data.py inputpars.txt
```

## Evaluation signals

- Regenerated vcdata.dat for R0=7.5 kpc matches the shipped sample file posdata_R75.dat / vcdata_R75.dat
- Regenerated vcdata.dat for R0=8.5 kpc matches the shipped sample files posdata_R85.dat / vcdata_R85.dat
- Default (baseline) run reproduces output/vcdata_baseline.dat and output/posdata_baseline.dat
- output/rawdata.dat contains the un-normalized original published values, confirming the conversion was applied to the right layer
- Peculiar-solar-motion correction on terminal velocities is small and varies sensibly with the assumed solar motion

## Limitations

- Raw data in galkin/data/data_*.dat and output/rawdata.dat are not normalized to the selected input parameters and must not be used directly unless fully aware of each original reference's assumptions and nomenclature
- Data from different references use different definitions and often overlap in observed sources, so the unified handling involves non-trivial assumptions
- wx is only needed in 'User-friendly window' mode and astropy only if coordinate conversion is done with astropy — the correction pipeline itself needs numpy

## Evidence

- [other] galkin processes the original data sets of the selected references and converts each data set consistently for the chosen values of the Galactic parameters: [evidence span withheld — anchor status: not_found]
- [other] the peculiar-solar-motion correction applied to terminal velocities being small and dependent on the assumed solar motion: [evidence span withheld — anchor status: not_found]
