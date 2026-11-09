---
name: positional-data-file-diff-verification
description: Use when validating that galkin_data.py reproduces its published positional
  data output for a specific Galactic parameter condition (e.g. R0=8.5 kpc) before
  trusting regenerated rotation-curve data in downstream analyses.
when_to_use_negative:
- 'When the goal is to inspect raw published measurements: use output/rawdata.dat
  or galkin/data/data_*.dat instead — the README warns these are ''given as in the
  original reference and thus not normalised to the selected input parameters'' and
  should not be used unless fully aware of each reference''s assumptions'
- When the target parameter set has no deposited reference sample file (only baseline,
  R85, and R75 reference samples are shipped) — no acceptance benchmark exists for
  other conditions
- When running galkin repeatedly on the fly for parameter scanning — use galkin_data_fast.py
  in-memory mode rather than file-based diff verification
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: galkin_data.py
  role: Retrieves and processes the original kinematic data and writes the normalized
    rotation curve and positional data to output/vcdata.dat and output/posdata.dat
    given inputpars.txt
  repo: github.com/galkintool/galkin
- name: galkin
  role: Python package providing the data handling routines that process the kinematic
    data under the chosen Galactic parameters
  repo: github.com/galkintool/galkin
- name: numpy
  role: Used to load the regenerated output/posdata.dat and reference posdata_R85.dat
    files for the row-by-row numeric comparison
- name: posdata_R85.dat
  role: Deposited sample reference output used as the benchmark for the R0=8.5 kpc
    condition
  repo: github.com/galkintool/galkin
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.softx.2016.12.006
    title: 'galkin: A new compilation of Milky Way rotation curve data'
  source_content_hash: sha256:ac5196f0646aa4139aad423ed459129f51097e41c6b77a791a448ce4bd435853
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - galkin
  - galkin_data.py
  - numpy
  - posdata_R85.dat
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# positional-data-file-diff-verification

## Summary

Regenerate the galkin positional data output (output/posdata.dat) for a chosen Galactic parameter set (e.g. R0=8.5 kpc) and verify it row by row against a deposited sample reference file (posdata_R85.dat) as a reproducibility check of galkin_data.py.

## When to use

Use when validating that galkin_data.py reproduces its published positional data output for a specific Galactic parameter condition (e.g. R0=8.5 kpc) before trusting regenerated rotation-curve data in downstream analyses.

## When NOT to use

- When the goal is to inspect raw published measurements: use output/rawdata.dat or galkin/data/data_*.dat instead — the README warns these are 'given as in the original reference and thus not normalised to the selected input parameters' and should not be used unless fully aware of each reference's assumptions
- When the target parameter set has no deposited reference sample file (only baseline, R85, and R75 reference samples are shipped) — no acceptance benchmark exists for other conditions
- When running galkin repeatedly on the fly for parameter scanning — use galkin_data_fast.py in-memory mode rather than file-based diff verification

## Inputs

- Input parameter file (inputpars.txt) setting R0=8.5 kpc with remaining baseline Galactic parameters at defaults
- galkin_data.py executable script from the galkin distribution (bin/)
- Deposited reference file posdata_R85.dat (sample position data for R0=8.5 kpc)
- Original kinematic data files galkin/data/data_*.dat

## Outputs

- Regenerated output/posdata.dat (normalized positional data)
- Regenerated output/vcdata.dat (normalized rotation curve data, for context)
- Verification report (e.g. posdata_R85_match_report.txt) stating whether posdata.dat matches posdata_R85.dat

## How to apply

Create an input parameter file setting R0=8.5 kpc with the remaining baseline Galactic parameters at defaults, then run 'python galkin_data.py inputpars.txt' so the code retrieves the original kinematic data and writes the normalized rotation curve and positional data to output/vcdata.dat and output/posdata.dat. Load the regenerated output/posdata.dat and the deposited reference bin/output/posdata_R85.dat (shipped as 'sample position data outputted by galkin_data.py for R0=8.5 kpc') with numpy. Compare the two files row by row, checking numeric columns against a tight tolerance and header lines as text, and record any differing rows. The acceptance criterion is a clean match — no differing rows — consistent with the README's statement that outputs 'can be compared to the files ... for checking purposes'.

## Related tools

- **galkin_data.py** (Retrieves and processes the original kinematic data and writes the normalized rotation curve and positional data to output/vcdata.dat and output/posdata.dat given inputpars.txt) — github.com/galkintool/galkin
- **galkin** (Python package providing the data handling routines that process the kinematic data under the chosen Galactic parameters) — github.com/galkintool/galkin
- **numpy** (Used to load the regenerated output/posdata.dat and reference posdata_R85.dat files for the row-by-row numeric comparison)
- **posdata_R85.dat** (Deposited sample reference output used as the benchmark for the R0=8.5 kpc condition) — github.com/galkintool/galkin

## Examples

```
python galkin_data.py inputpars.txt; python -c "import numpy as np; a=np.loadtxt('output/posdata.dat'); b=np.loadtxt('posdata_R85.dat'); print(np.allclose(a,b))"
```

## Evaluation signals

- The verification report (posdata_R85_match_report.txt) states a clean match: zero differing rows between regenerated posdata.dat and posdata_R85.dat
- Numeric data columns agree within the tight tolerance used in the row-by-row numpy comparison
- Header lines of the two files match as text
- The comparison was run against the correct deposited reference file for the chosen parameter condition (posdata_R85.dat for R0=8.5 kpc, not posdata_baseline.dat or posdata_R75.dat)

## Limitations

- The raw data outputs (output/rawdata.dat and galkin/data/data_*.dat) are given as in the original references and are NOT normalised to the selected input parameters; only the normalized vcdata.dat/posdata.dat should be diff-verified
- Deposited reference sample files exist only for the baseline parameter set, R0=8.5 kpc, and R0=7.5 kpc; other parameter conditions lack benchmarks
- The code runs under Python 2.7.6 (or higher) and requires matplotlib, numpy, wx (window mode) and astropy (optional coordinate conversion) — environment must satisfy these dependencies
- Data files under galkin/data/ contain original published data per reference with differing Galactocentric parameters and definitions, so they must not be compared directly

## Evidence
