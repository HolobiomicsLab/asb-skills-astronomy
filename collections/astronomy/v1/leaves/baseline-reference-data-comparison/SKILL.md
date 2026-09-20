---
name: baseline-reference-data-comparison
description: Use when galkin has just been installed or reconfigured and you need
  to verify that galkin_data.py with the default (baseline) Galactic parameters reproduces
  the reference rotation-curve and tracer position outputs shipped in the distribution.
when_to_use_negative:
- You are running galkin_data.py with non-default Galactic parameters (e.g. R0=8.5
  or 7.5 kpc) — the baseline files are only valid for the default setup, so a diff
  against them will fail by design.
- You intend to analyse galkin/data/data_*.dat or rawdata.dat directly — the README
  warns these are original published data not normalised to the selected input parameters
  and should not be used unless all per-reference assumptions are understood.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: galkin_data.py
  role: Generates the output/vcdata.dat and output/posdata.dat files that are compared
    against the baseline references
  repo: github.com/galkintool/galkin
- name: galkin
  role: Python package whose data-handling routines produce the outputs under validation
  repo: github.com/galkintool/galkin
- name: galkin_plotter.py
  role: Plots the output data so the rendered rotation curve and tracer spatial distribution
    can be visually checked against baseline expectations
  repo: github.com/galkintool/galkin
- name: Python
  role: Runtime for installing and executing the galkin scripts (python 2.7.6 or higher)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.softx.2016.12.006
    title: 'galkin: A new compilation of Milky Way rotation curve data'
  source_content_hash: sha256:e3896d7aa8f35e721cdbbbcbbcab47b6cb636238e03b00ec6ca6c90f9a2c3fdd
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Python
  - galkin
  - galkin_data.py
  - galkin_plotter.py
  - matplotlib
  - numpy
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# baseline-reference-data-comparison

## Summary

Validate a galkin installation and parameter setup by diffing the code's output data files against the shipped baseline reference files (vcdata_baseline.dat, posdata_baseline.dat) produced for the default Galactic parameter configuration.

## When to use

Use when galkin has just been installed or reconfigured and you need to verify that galkin_data.py with the default (baseline) Galactic parameters reproduces the reference rotation-curve and tracer position outputs shipped in the distribution.

## When NOT to use

- You are running galkin_data.py with non-default Galactic parameters (e.g. R0=8.5 or 7.5 kpc) — the baseline files are only valid for the default setup, so a diff against them will fail by design.
- You intend to analyse galkin/data/data_*.dat or rawdata.dat directly — the README warns these are original published data not normalised to the selected input parameters and should not be used unless all per-reference assumptions are understood.

## Inputs

- output/vcdata.dat (rotation-curve data produced by galkin_data.py for the default parameter setup)
- output/posdata.dat (positional data produced by galkin_data.py for the default parameter setup)
- output/vcdata_baseline.dat and output/posdata_baseline.dat (shipped reference files in bin/output/)
- output/rawdata.dat (optional raw data output to check against rawdata_baseline.dat)

## Outputs

- Pass/fail comparison of vcdata.dat vs vcdata_baseline.dat and posdata.dat vs posdata_baseline.dat
- Optionally rendered figures from galkin_plotter.py whose plotted points/curve are consistent with the baseline reference data

## How to apply

After installing the package with 'python setup.py install', run galkin_data.py with the default parameter setup (either the user-friendly window or inputpars.txt) so it writes output/vcdata.dat and output/posdata.dat. Then compare these files file-by-file against the shipped reference files output/vcdata_baseline.dat and output/posdata_baseline.dat, checking that the rotation-curve values and tracer positional data match for the baseline configuration. Optionally also generate output/rawdata.dat and confirm it corresponds to rawdata_baseline.dat, keeping in mind the raw data are un-normalised to the selected input parameters. If the baseline files differ unexpectedly, suspect an installation, dependency, or parameter-file problem before trusting subsequent scans (e.g. via galkin_data_fast.py).

## Related tools

- **galkin_data.py** (Generates the output/vcdata.dat and output/posdata.dat files that are compared against the baseline references) — github.com/galkintool/galkin
- **galkin** (Python package whose data-handling routines produce the outputs under validation) — github.com/galkintool/galkin
- **galkin_plotter.py** (Plots the output data so the rendered rotation curve and tracer spatial distribution can be visually checked against baseline expectations) — github.com/galkintool/galkin
- **Python** (Runtime for installing and executing the galkin scripts (python 2.7.6 or higher))

## Examples

```
python galkin_data.py inputpars.txt && python galkin_plotter.py output/vcdata.dat output/posdata.dat  # then diff output/vcdata.dat output/vcdata_baseline.dat
```

## Evaluation signals

- output/vcdata.dat matches output/vcdata_baseline.dat (line-by-line/numeric diff) for the default parameter setup
- output/posdata.dat matches output/posdata_baseline.dat for the default parameter setup
- Figures rendered by 'python galkin_plotter.py output/vcdata.dat output/posdata.dat' exist, are readable images, and their plotted points/curve are consistent with the baseline reference files
- Rerunning galkin_data.py with identical inputs reproduces identical output files (deterministic behaviour)

## Limitations

- The raw data (output/rawdata.dat and galkin/data/data_*.dat) are given as in the original references and are not normalised to the selected input parameters; the README explicitly cautions against using them without full awareness of each reference's assumptions and nomenclature.
- Baseline comparison is only meaningful for the default Galactic parameter setup; alternative sample outputs exist for R0=8.5 kpc and R0=7.5 kpc.
- The code requires python 2.7.6 (or higher) with matplotlib, numpy, and (for the window mode) wx, so environment drift can cause spurious baseline mismatches.

## Evidence

- [readme] For the baseline Galactic parameter setup (default), these files can be compared to the files output/vcdata_baseline.dat and output/posdata_baseline.dat for checking purposes.: "For the baseline Galactic parameter setup (default), these files can be compared to the files output/vcdata_baseline.dat and output/posdata_baseline.dat for checking purposes."
- [methods] The code outputs rotation curve and positional data in the files output/vcdata.dat and output/posdata.dat, respectively: "The code outputs rotation curve and positional data in the files output/vcdata.dat and output/posdata.dat, respectively"
- [readme] Notice however that these raw data are given as in the original reference and thus not normalised to the selected input parameters.: "Notice however that these raw data are given as in the original reference and thus not normalised to the selected input parameters."
- [readme] Please do not use these files directly unless you are fully aware of all assumptions of each reference.: "Please do not use these files directly unless you are fully aware of all assumptions of each reference."
