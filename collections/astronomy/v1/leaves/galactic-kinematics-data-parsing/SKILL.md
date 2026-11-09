---
name: galactic-kinematics-data-parsing
description: Use when you need to quantify the composition of the galkin Milky Way
  rotation-curve compilation by counting tracer measurements per category (gas, stars,
  masers) from the raw data file.
when_to_use_negative:
- 'You need Galactocentric-parameter-normalised rotation-curve values: rawdata.dat
  is given as in the original references and not normalised to the selected input
  parameters, so use output/vcdata.dat instead'
- You plan to use galkin/data/data_*.dat or rawdata.dat directly for scientific analysis
  without being fully aware of the assumptions and nomenclature of each original reference
  (the README caveat)
- You need repeated in-memory data processing for parameter scanning rather than a
  one-off file parse — use galkin_data_fast.py instead
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3332
tools:
- name: galkin
  role: Python package whose data handling routines (galkin/processdata.py) import
    and process the original kinematic data being parsed
  repo: github.com/galkintool/galkin
- name: galkin_data.py
  role: Script run under the baseline parameter setup to generate output/rawdata.dat,
    the raw measurement file that this skill parses
  repo: github.com/galkintool/galkin
- name: numpy
  role: Used to parse the tracer and source/reference columns of each measurement
    record and tabulate per-category counts
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.softx.2016.12.006
    title: 'galkin: A new compilation of Milky Way rotation curve data'
  source_content_hash: sha256:0a029e40c8dff77ea44e29fd15f79ab4af939425de53178cd9dd2a05b3b2c4f7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - galkin
  - galkin_data.py
  - numpy
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3332
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# galactic-kinematics-data-parsing

## Summary

Parse the raw rotation-curve measurement records in galkin's output/rawdata.dat, classify each kinematic tracer measurement into the gas, stars, or masers categories, and validate the tabulated counts against the compilation's reported total of 2780 measurements.

## When to use

Use when you need to quantify the composition of the galkin Milky Way rotation-curve compilation by counting tracer measurements per category (gas, stars, masers) from the raw data file.

## When NOT to use

- You need Galactocentric-parameter-normalised rotation-curve values: rawdata.dat is given as in the original references and not normalised to the selected input parameters, so use output/vcdata.dat instead
- You plan to use galkin/data/data_*.dat or rawdata.dat directly for scientific analysis without being fully aware of the assumptions and nomenclature of each original reference (the README caveat)
- You need repeated in-memory data processing for parameter scanning rather than a one-off file parse — use galkin_data_fast.py instead

## Inputs

- output/rawdata.dat — raw kinematic data in original references, produced by galkin_data.py under the baseline Galactic parameter setup
- galkin/data/data_*.dat — original kinematic data files retrieved from the source references in Table 1 of the manuscript

## Outputs

- tracer_measurement_counts.csv — per-category counts of measurements for gas, stars, and masers
- total measurement count checked against the reported 2780

## How to apply

Generate the raw data product by running galkin_data.py under the baseline Galactic parameter setup, which writes output/rawdata.dat containing the original published measurements. Parse each measurement record's tracer and source/reference columns using numpy under Python 2.7.6 (or higher). Assign each record to one of the three tracer categories used by the compilation — gas kinematics, stellar kinematics, or masers — and tabulate per-category counts with numpy counting routines. Sum the per-category counts and check the total equals 2780, the reported size of the compilation spanning almost four decades of literature and optimised for Galactocentric radii between 3 and 20 kpc. Write the per-category counts to tracer_measurement_counts.csv as the final deliverable.

## Related tools

- **galkin** (Python package whose data handling routines (galkin/processdata.py) import and process the original kinematic data being parsed) — github.com/galkintool/galkin
- **galkin_data.py** (Script run under the baseline parameter setup to generate output/rawdata.dat, the raw measurement file that this skill parses) — github.com/galkintool/galkin
- **numpy** (Used to parse the tracer and source/reference columns of each measurement record and tabulate per-category counts)

## Examples

```
python galkin_data.py inputpars.txt  # baseline setup produces output/rawdata.dat for tracer-category parsing
```

## Evaluation signals

- The three per-category counts (gas, stars, masers) sum exactly to 2780, the reported total of the compilation
- Every parsed record is assigned to exactly one of the three tracer categories — no unclassified or duplicated records
- Per-category counts are consistent with the rawdata_baseline_gas.dat, rawdata_baseline_stars.dat, and rawdata_baseline_masers.dat sample outputs shipped with the distribution
- tracer_measurement_counts.csv contains exactly three category rows matching the gas/stars/masers nomenclature of the compilation

## Limitations

- rawdata.dat entries are given as in the original reference and are not normalised to the selected input Galactocentric parameters (e.g. R0)
- Data from different references use different Galactocentric parameters, different definitions, and often overlap in observed sources, so raw records cannot be naively merged or compared
- The original data files under galkin/data/ should not be used directly unless fully aware of all assumptions of each reference

## Evidence

- [abstract] optimised to Galactocentric radii between 3 and 20 kpc and includes the kinematics of gas, stars and masers in a total of 2780 measurements carefully collected from almost four decades of literature: "optimised to Galactocentric radii between 3 and 20 kpc and includes the kinematics of gas, stars and masers in a total of 2780 measurements"
- [readme] Notice however that these raw data are given as in the original reference and thus not normalised to the selected input parameters.: "Notice however that these raw data are given as in the original reference and thus not normalised to the selected input parameters."
