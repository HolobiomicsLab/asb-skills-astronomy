---
name: galactic-tracer-data-normalization
description: Use when you need rotation-curve position and velocity data re-parameterised
  to a chosen Galactocentric parameter set (e.g. R0), starting from galkin's original
  data_*.dat files, and want output checkable against deposited baseline references.
when_to_use_negative:
- You need the original published values of a specific reference as-is — use galkin/data/data_*.dat
  or output/rawdata.dat directly only if fully aware of all assumptions and nomenclature
  of each original reference.
- Your goal is only to visualise existing galkin output rather than regenerate normalized
  data — use galkin_plotter.py on existing output/vcdata.dat and output/posdata.dat
  instead.
- You are scanning many parameter sets repeatedly and want no file I/O overhead —
  use galkin_data_fast.py, which processes data in-memory.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3318
tools:
- name: galkin
  role: Python package whose routines import and process the original kinematic data,
    normalising measurements to the selected input Galactic parameters
  repo: github.com/galkintool/galkin
- name: galkin_data.py
  role: Script that retrieves the data given an input parameter file (or user-friendly
    window) and writes normalized vcdata.dat and posdata.dat outputs
  repo: github.com/galkintool/galkin
- name: galkin_data_fast.py
  role: In-memory variant of the data retriever for repeated parameter scanning without
    writing input/output files
  repo: github.com/galkintool/galkin
- name: numpy
  role: Used to load and row-by-row compare the regenerated posdata.dat against the
    deposited reference file (e.g. posdata_R85.dat)
- name: Python
  role: Runtime environment; the code runs under Python 2.7.6 (or higher) with matplotlib,
    numpy, wx and optionally astropy
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.softx.2016.12.006
    title: 'galkin: A new compilation of Milky Way rotation curve data'
  source_content_hash: sha256:928eaee688b3b6539cb7a8fb0c1588d3d46683cdf62d8d4b30aaa4751c745143
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
  - galkin
  - galkin_data.py
  - galkin_data_fast.py
  - numpy
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# galactic-tracer-data-normalization

## Summary

Re-normalizes the original Milky Way rotation-curve kinematic data (gas, stars, masers) compiled by galkin to a user-chosen set of Galactic parameters (e.g. R0=8.5 kpc), producing normalized rotation-curve (output/vcdata.dat) and positional (output/posdata.dat) files that can be validated against deposited baseline sample files.

## When to use

Use when you need rotation-curve position and velocity data re-parameterised to a chosen Galactocentric parameter set (e.g. R0), starting from galkin's original data_*.dat files, and want output checkable against deposited baseline references.

## When NOT to use

- You need the original published values of a specific reference as-is — use galkin/data/data_*.dat or output/rawdata.dat directly only if fully aware of all assumptions and nomenclature of each original reference.
- Your goal is only to visualise existing galkin output rather than regenerate normalized data — use galkin_plotter.py on existing output/vcdata.dat and output/posdata.dat instead.
- You are scanning many parameter sets repeatedly and want no file I/O overhead — use galkin_data_fast.py, which processes data in-memory.

## Inputs

- input parameter file (inputpars.txt) specifying Galactic parameters such as R0=8.5 kpc
- original kinematic data files galkin/data/data_*.dat retrieved from source references in Table 1 of the manuscript
- deposited reference sample files (e.g. posdata_baseline.dat, posdata_R85.dat, vcdata_baseline.dat) for comparison

## Outputs

- output/vcdata.dat — normalized rotation curve data
- output/posdata.dat — normalized positional data
- output/rawdata.dat — raw data as in the original references (not normalized to selected input parameters)
- row-by-row comparison/match report between regenerated and deposited reference files

## How to apply

Write an input parameter file (e.g. inputpars.txt) specifying the desired Galactic parameters, such as R0=8.5 kpc, leaving the remaining baseline parameters at defaults, then run 'python galkin_data.py inputpars.txt' so galkin_data.py retrieves and processes the original kinematic data in galkin/data/data_*.dat. The code writes the normalized rotation curve to output/vcdata.dat and the positional data to output/posdata.dat, and additionally the raw data in the original references to output/rawdata.dat. Validate the regenerated output by loading output/posdata.dat (or output/vcdata.dat) with numpy together with the corresponding deposited sample file (e.g. posdata_R85.dat for R0=8.5 kpc, or posdata_baseline.dat for the baseline set) and comparing numeric columns row by row within a tight tolerance, with header lines compared as text. Do not use the raw data files directly unless fully aware of each original reference's assumptions and nomenclature, since raw data are not normalized to the selected input parameters. For repeated on-the-fly parameter scanning, use galkin_data_fast.py to process the data in-memory without writing input/output files.

## Related tools

- **galkin** (Python package whose routines import and process the original kinematic data, normalising measurements to the selected input Galactic parameters) — github.com/galkintool/galkin
- **galkin_data.py** (Script that retrieves the data given an input parameter file (or user-friendly window) and writes normalized vcdata.dat and posdata.dat outputs) — github.com/galkintool/galkin
- **galkin_data_fast.py** (In-memory variant of the data retriever for repeated parameter scanning without writing input/output files) — github.com/galkintool/galkin
- **numpy** (Used to load and row-by-row compare the regenerated posdata.dat against the deposited reference file (e.g. posdata_R85.dat))
- **Python** (Runtime environment; the code runs under Python 2.7.6 (or higher) with matplotlib, numpy, wx and optionally astropy)

## Examples

```
python galkin_data.py inputpars.txt
```

## Evaluation signals

- The verification report (e.g. posdata_R85_match_report.txt) states whether output/posdata.dat matches the deposited reference file for the chosen parameter set; a clean match (no differing rows) is the acceptance criterion.
- For the default baseline Galactic parameter setup, output/vcdata.dat and output/posdata.dat match the shipped sample files vcdata_baseline.dat and posdata_baseline.dat.
- Numeric columns of regenerated vs reference files agree within a tight tolerance row by row; header lines match as text.
- All three expected outputs (vcdata.dat, posdata.dat, rawdata.dat) are present in the output/ directory after running galkin_data.py.

## Limitations

- Raw data output (output/rawdata.dat) and the galkin/data/data_*.dat files are given as in the original references and are NOT normalized to the selected input parameters; do not use them directly unless fully aware of all assumptions and nomenclature of each original reference.
- References use different Galactocentric parameters and different definitions, and often overlap in observed sources, so direct use of raw files can be misleading.
- The wx package is only required in 'User-friendly window' mode and astropy only if coordinate conversion with astropy is requested; the code targets Python 2.7.6 or higher.
- The compilation is optimised for Galactocentric radii between 3 and 20 kpc.
