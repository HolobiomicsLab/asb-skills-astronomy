---
name: reference-output-consistency-checking
description: Use when a tool is supposed to reproduce known reference outputs (e.g.
  after re-running a conversion under new Galactic parameters R0=7.5 or 8.5 kpc) and
  you must confirm the regenerated files match shipped samples.
when_to_use_negative:
- When validating the raw data files galkin/data/data_*.dat or output/rawdata.dat
  directly — these are given as in the original references and are deliberately NOT
  normalised to the selected input parameters, so comparison against converted reference
  outputs is invalid.
- When the goal is a new scientific measurement rather than correctness of the tool's
  parameter-frame conversion — baseline/regression comparison adds no information
  there.
- When no shipped reference output exists for the chosen parameter set (e.g. an arbitrary
  non-sample R0), since there is nothing authoritative to compare against.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: galkin_data.py
  role: Retrieves and outputs the data for a user-defined Galactic parameter set;
    its outputs are the files compared against the shipped samples.
  repo: github.com/galkintool/galkin
- name: galkin
  role: Package whose data-handling routines perform the parameter-frame conversion
    being validated.
  repo: github.com/galkintool/galkin
- name: Python
  role: Runtime used to install and run galkin_data.py (Python 2.7.6 or higher).
- name: numpy
  role: Required Python dependency for running the galkin package.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.softx.2016.12.006
    title: 'galkin: A new compilation of Milky Way rotation curve data'
  source_content_hash: sha256:4faf3589edd49082c293fd06a8800723613429cdaf2f73c9498ce513283b7c23
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
  - galkin
  - galkin_data.py
  - numpy
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# Reference-output consistency checking

## Summary

Validates that a scientific tool's reprocessed outputs match shipped reference/sample output files by direct comparison — here, galkin's regenerated vcdata/posdata files are diffed against bundled R0=7.5 kpc, R0=8.5 kpc, and baseline sample files.

## When to use

Use when a tool is supposed to reproduce known reference outputs (e.g. after re-running a conversion under new Galactic parameters R0=7.5 or 8.5 kpc) and you must confirm the regenerated files match shipped samples.

## When NOT to use

- When validating the raw data files galkin/data/data_*.dat or output/rawdata.dat directly — these are given as in the original references and are deliberately NOT normalised to the selected input parameters, so comparison against converted reference outputs is invalid.
- When the goal is a new scientific measurement rather than correctness of the tool's parameter-frame conversion — baseline/regression comparison adds no information there.
- When no shipped reference output exists for the chosen parameter set (e.g. an arbitrary non-sample R0), since there is nothing authoritative to compare against.

## Inputs

- input parameter file (inputpars.txt) specifying the Galactic parameter set, e.g. R0=7.5 kpc or R0=8.5 kpc with peculiar solar motion
- original kinematic data files galkin/data/data_*.dat from source references in Table 1
- shipped reference sample output files (posdata_R75.dat, vcdata_R75.dat, posdata_R85.dat, vcdata_R85.dat, output/vcdata_baseline.dat, output/posdata_baseline.dat)

## Outputs

- regenerated rotation-curve data file (output/vcdata.dat)
- regenerated positional data file (output/posdata.dat)
- pass/fail comparison result per parameter set (R0=7.5 and R0=8.5 kpc) against the corresponding reference sample files

## How to apply

Run the tool's data processing pipeline with an explicit parameter choice and capture its outputs. In galkin's case, run 'python galkin_data.py inputpars.txt' with the input parameter file set to R0=7.5 kpc (including the peculiar-solar-motion choice) and compare the generated rotation-curve and positional outputs against the shipped sample files posdata_R75.dat and vcdata_R75.dat; repeat with R0=8.5 kpc against posdata_R85.dat and vcdata_R85.dat. As an additional baseline check, run the default parameter setup and compare against output/vcdata_baseline.dat and output/posdata_baseline.dat. Success is judged per-file: each regenerated vcdata/posdata file for a given R0 must match the corresponding shipped sample file for that R0 value. The rationale is that the tool converts original kinematic data sets (galkin/data/data_*.dat) consistently into the chosen parameter frame, so any deviation from the reference samples indicates a conversion or configuration error.

## Related tools

- **galkin_data.py** (Retrieves and outputs the data for a user-defined Galactic parameter set; its outputs are the files compared against the shipped samples.) — github.com/galkintool/galkin
- **galkin** (Package whose data-handling routines perform the parameter-frame conversion being validated.) — github.com/galkintool/galkin
- **Python** (Runtime used to install and run galkin_data.py (Python 2.7.6 or higher).)
- **numpy** (Required Python dependency for running the galkin package.)

## Examples

```
python galkin_data.py inputpars.txt && diff output/vcdata.dat vcdata_R75.dat
```

## Evaluation signals

- Regenerated vcdata/posdata files for R0=7.5 kpc match the shipped posdata_R75.dat and vcdata_R75.dat sample files.
- Regenerated vcdata/posdata files for R0=8.5 kpc match the shipped posdata_R85.dat and vcdata_R85.dat sample files.
- Default (baseline) run matches output/vcdata_baseline.dat and output/posdata_baseline.dat.
- Output/rawdata.dat contains the original, unnormalised reference data — if converted values appear in the raw file, the conversion step was skipped or misconfigured.
- The peculiar-solar-motion correction applied to terminal velocities is small; large shifts in gas terminal velocities suggest an incorrect solar-motion parameter choice.

## Limitations

- The sample files cover only specific parameter sets (baseline, R0=7.5 kpc, R0=8.5 kpc); consistency checking cannot validate arbitrary parameter choices without a reference.
- rawdata.dat and galkin/data/data_*.dat are given as in the original references and not normalised to the selected input parameters; using them directly without understanding each reference's assumptions and nomenclature is explicitly warned against.
- The check confirms bit-level/expected reproducibility of the tool's conversion, not the scientific correctness of the underlying published measurements.

## Evidence

- [results] galkin processes the original data sets of the selected references and converts each data set consistently for the chosen values of the Galactic parameters: [evidence span withheld — anchor status: not_found]
