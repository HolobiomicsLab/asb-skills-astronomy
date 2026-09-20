---
name: kinematic-tracer-data-retrieval
description: Use when you need a unified, parameter-normalized compilation of Galactic
  rotation-curve tracer measurements (gas, stars, masers) for Galactocentric radii
  ~3–20 kpc, either from files or in-memory for parameter scans.
when_to_use_negative:
- You need raw published values from a specific reference — use galkin/data/data_*.dat
  or output/rawdata.dat only if fully aware of each reference's assumptions and nomenclature,
  since these are not normalized
- You need rotation-curve data outside the ~3–20 kpc Galactocentric range the compilation
  was optimized for
- You only need plotting of already-generated output files — call galkin_plotter.py
  directly on existing output/vcdata.dat and output/posdata.dat instead of re-running
  retrieval
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3264
tools:
- name: galkin
  role: Python package with data-handling routines (processdata.py, readparsFile.py,
    functions.py) that imports and normalizes the original kinematic data; loadable
    into external codes
  repo: github.com/galkintool/galkin
- name: galkin_data.py
  role: Script that retrieves and outputs the rotation-curve, positional, and raw
    data given input parameters (window or file mode)
  repo: github.com/galkintool/galkin
- name: galkin_data_fast.py
  role: In-memory variant of the retriever for repeated on-the-fly use, e.g. scanning
    over input parameters, without file I/O
  repo: github.com/galkintool/galkin
- name: Python 2.7.6
  role: Runtime environment (or higher) under which the galkin package and scripts
    are executed
- name: numpy
  role: Required Python package dependency for running galkin
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.softx.2016.12.006
    title: 'galkin: A new compilation of Milky Way rotation curve data'
  source_content_hash: sha256:fa99e9b8d8adf0c9e7cdf4b062e3c17656b52e644f4230bdaafe0465150c8375
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python 2.7.6
  - galkin
  - galkin_data.py
  - galkin_data_fast.py
  - numpy
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3264
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# kinematic-tracer-data-retrieval

## Summary

Retrieve and normalize Milky Way rotation-curve measurements (gas, stars, masers; 2780 total) using galkin_data.py, which unifies data from references using different Galactocentric parameters into normalized vcdata.dat and posdata.dat outputs.

## When to use

Use when you need a unified, parameter-normalized compilation of Galactic rotation-curve tracer measurements (gas, stars, masers) for Galactocentric radii ~3–20 kpc, either from files or in-memory for parameter scans.

## When NOT to use

- You need raw published values from a specific reference — use galkin/data/data_*.dat or output/rawdata.dat only if fully aware of each reference's assumptions and nomenclature, since these are not normalized
- You need rotation-curve data outside the ~3–20 kpc Galactocentric range the compilation was optimized for
- You only need plotting of already-generated output files — call galkin_plotter.py directly on existing output/vcdata.dat and output/posdata.dat instead of re-running retrieval

## Inputs

- input parameter file (inputpars.txt format) specifying Galactic parameters, or interactive window input
- galkin/data/data_*.dat original kinematic data files from Table 1 source references
- reference baseline outputs vcdata_baseline.dat and posdata_baseline.dat for checking

## Outputs

- output/vcdata.dat — normalized rotation curve data
- output/posdata.dat — normalized positional data
- output/rawdata.dat — raw un-normalized data as in the original references
- in-memory measurement arrays when using galkin_data_fast.py

## How to apply

Install the galkin package with 'python setup.py install' under Python 2.7.6 or higher (requires matplotlib, numpy; wx for window mode, astropy for coordinate conversion). Run 'python galkin_data.py inputpars.txt' to feed the code the baseline parameter specifications from the provided inputpars.txt example, or invoke it with no argument for the user-friendly window mode. The script processes the original kinematic data under galkin/data/data_*.dat (sourced from Table 1 of the manuscript) and writes the normalized rotation curve, positional, and raw data to output/vcdata.dat, output/posdata.dat, and output/rawdata.dat. For repeated on-the-fly use such as scanning over input parameters, use galkin_data_fast.py, which processes data in memory without writing input/output files. Validate a run by diffing the generated files against the deposited baselines output/vcdata_baseline.dat and output/posdata_baseline.dat; expect 2780 combined gas, star, and maser measurements, and optionally per-tracer baselines (e.g. *_baseline_gas.dat) or R0 variants (R85/R75).

## Related tools

- **galkin** (Python package with data-handling routines (processdata.py, readparsFile.py, functions.py) that imports and normalizes the original kinematic data; loadable into external codes) — github.com/galkintool/galkin
- **galkin_data.py** (Script that retrieves and outputs the rotation-curve, positional, and raw data given input parameters (window or file mode)) — github.com/galkintool/galkin
- **galkin_data_fast.py** (In-memory variant of the retriever for repeated on-the-fly use, e.g. scanning over input parameters, without file I/O) — github.com/galkintool/galkin
- **Python 2.7.6** (Runtime environment (or higher) under which the galkin package and scripts are executed)
- **numpy** (Required Python package dependency for running galkin)

## Examples

```
python galkin_data.py inputpars.txt
```

## Evaluation signals

- Regenerated output/vcdata.dat and output/posdata.dat match the deposited reference files output/vcdata_baseline.dat and output/posdata_baseline.dat for the baseline Galactic parameter set
- Combined gas, star, and maser measurements total 2780, consistent with the galkin compilation
- output/rawdata.dat reflects the un-normalized original references (only expected if the user is aware of each reference's assumptions)
- Per-tracer runs match the corresponding baselines, e.g. vcdata_baseline_gas.dat, vcdata_baseline_stars.dat, vcdata_baseline_masers.dat
- Alternative R0 choices reproduce the shipped variants, e.g. posdata_R85.dat (R0=8.5 kpc) and posdata_R75.dat (R0=7.5 kpc)

## Limitations

- Raw data in output/rawdata.dat and files under galkin/data/ are given as in the original references and are NOT normalized to the selected input parameters; do not use them directly unless fully aware of all assumptions and nomenclature of each original reference
- The compilation unifies sources using different Galactocentric parameters, different definitions, and often overlapping observed sources, so cross-reference provenance must be kept in mind
- wx is only required for 'User-friendly window' mode and astropy only if the user wants coordinate conversion done with astropy; running in file mode does not need them

## Evidence
