---
name: galactic-rotation-curve-data-compilation
description: Use when a Milky Way rotation curve normalized to a chosen Galactocentric
  parameter set is needed, or when reproducing/validating the 2780-measurement galkin
  compilation between ~3 and 20 kpc.
when_to_use_negative:
- 'Do not use galkin/data/data_*.dat or output/rawdata.dat directly: they contain
  original published data not normalized to the selected input parameters and should
  only be used if fully aware of each reference''s assumptions and nomenclature.'
- The baseline validation files are specific to the default Galactic parameter setup;
  they will not match runs with altered parameters such as R0=8.5 or 7.5 kpc.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3315
- http://edamontology.org/topic_3452
tools:
- name: galkin
  role: Python package providing the data handling routines that import and process
    the original kinematic data
  repo: github.com/galkintool/galkin
- name: galkin_data.py
  role: Data retriever script that reads the input parameters and writes vcdata.dat,
    posdata.dat and rawdata.dat
  repo: github.com/galkintool/galkin
- name: galkin_data_fast.py
  role: In-memory retriever for repeated on-the-fly use, e.g. scanning over input
    parameters
  repo: github.com/galkintool/galkin
- name: galkin_plotter.py
  role: Ready-to-use plotter for the vcdata.dat and posdata.dat outputs
  repo: github.com/galkintool/galkin
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.softx.2016.12.006
    title: 'galkin: A new compilation of Milky Way rotation curve data'
  source_content_hash: sha256:406d4b6244a2c36cc2cf1663322ce4c8dd920784d0f31805a10a58da5d2aed69
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - galkin
  - galkin_data.py
  - galkin_data_fast.py
  - galkin_plotter.py
  - numpy
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3315
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# galactic-rotation-curve-data-compilation

## Summary

Retrieve and normalize the galkin compilation of 2780 Milky Way rotation-curve measurements (gas, stars, masers) using galkin_data.py, and verify the run against deposited baseline reference outputs for the default Galactic parameter set.

## When to use

Use when a Milky Way rotation curve normalized to a chosen Galactocentric parameter set is needed, or when reproducing/validating the 2780-measurement galkin compilation between ~3 and 20 kpc.

## When NOT to use

- Do not use galkin/data/data_*.dat or output/rawdata.dat directly: they contain original published data not normalized to the selected input parameters and should only be used if fully aware of each reference's assumptions and nomenclature.
- The baseline validation files are specific to the default Galactic parameter setup; they will not match runs with altered parameters such as R0=8.5 or 7.5 kpc.

## Inputs

- Example input parameter file inputpars.txt (baseline Galactic parameter specifications)
- Original kinematic data files galkin/data/data_*.dat (from source references in Table 1 of the manuscript)

## Outputs

- output/vcdata.dat (normalized rotation curve)
- output/posdata.dat (normalized positional data)
- output/rawdata.dat (raw data as in the original references, not normalized)

## How to apply

Install the galkin package with 'python setup.py install' under Python 2.7.6 (or higher), with matplotlib and numpy required, and wx/astropy only in window mode or for astropy-based coordinate conversion. Run the data retriever in input-file mode, 'python galkin_data.py inputpars.txt', so it processes the original kinematic data under galkin/data/data_*.dat (sourced from Table 1 of the manuscript) and writes the normalized rotation curve, positional, and raw data to output/vcdata.dat, output/posdata.dat, and output/rawdata.dat. Compare the regenerated vcdata.dat and posdata.dat against the deposited reference files output/vcdata_baseline.dat and output/posdata_baseline.dat, which are sample outputs of galkin_data.py for the baseline parameter set. Optionally plot the outputs with 'python galkin_plotter.py output/vcdata.dat output/posdata.dat'. For repeated on-the-fly parameter scanning, use galkin_data_fast.py, which processes data in memory without writing I/O files.

## Related tools

- **galkin** (Python package providing the data handling routines that import and process the original kinematic data) — github.com/galkintool/galkin
- **galkin_data.py** (Data retriever script that reads the input parameters and writes vcdata.dat, posdata.dat and rawdata.dat) — github.com/galkintool/galkin
- **galkin_data_fast.py** (In-memory retriever for repeated on-the-fly use, e.g. scanning over input parameters) — github.com/galkintool/galkin
- **galkin_plotter.py** (Ready-to-use plotter for the vcdata.dat and posdata.dat outputs) — github.com/galkintool/galkin

## Examples

```
python galkin_data.py inputpars.txt && python galkin_plotter.py output/vcdata.dat output/posdata.dat
```

## Evaluation signals

- Regenerated output/vcdata.dat matches the deposited reference output/vcdata_baseline.dat for the baseline parameter set.
- Regenerated output/posdata.dat matches the deposited reference output/posdata_baseline.dat.
- The compiled dataset totals 2780 gas, star, and maser measurements spanning Galactocentric radii between 3 and 20 kpc.
- rawdata.dat reproduces the un-normalized original published values (as in the source references), unlike the normalized vc/posdata files.
- galkin_plotter.py accepts the outputs without error: 'python galkin_plotter.py output/vcdata.dat output/posdata.dat'.

## Limitations

- The code runs under Python 2.7.6 (or higher) and requires matplotlib, numpy, and wx (window mode) / astropy (optional coordinate conversion).
- The tool unifies data from references with different Galactocentric parameters and definitions, often overlapping in observed sources; raw data files must not be used without awareness of each reference's assumptions.
- The baseline check only validates the default Galactic parameter setup, not arbitrary parameter choices.

## Evidence
