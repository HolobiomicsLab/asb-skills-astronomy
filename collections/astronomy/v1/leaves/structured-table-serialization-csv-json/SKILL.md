---
name: structured-table-serialization-csv-json
description: Use when a task asks to reproduce a published parameter table — e.g.
  bar pattern speed 37 km/s/kpc, two-armed spiral 13.1 km/s/kpc (~25% contrast), three-armed
  16.4 km/s/kpc (~9% contrast) — as a machine-readable CSV/JSON artifact.
when_to_use_negative:
- When the goal is to re-run the SPIBACK backward-integration model itself (spiback_minimalversion_run.ipynb)
  rather than tabulate its already-published fiducial parameters
- When the parameters already exist in a structured, machine-readable format (e.g.
  a released data table) — re-parsing prose would risk transcription error
- When values must be recomputed from Gaia DR3 velocity maps instead of quoted from
  the paper's reported fiducial model
tools:
- name: Python 3.10
  role: Runs the parsing and serialization script; the workflow specifies tabulating
    parameters with pandas/Python's csv module
- name: Jupyter Notebook
  role: Environment in which the extraction-and-serialization workflow is executed
    and the output table checked
- name: SPIBACK
  role: Source of the fiducial model parameters being tabulated; its default settings
    correspond to the fiducial model described in Khalil et al.
  repo: github.com/yrkhalil/SPIBACK
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202453077
    title: A non-axisymmetric potential for the Milky Way disk
  source_content_hash: sha256:1fa35e6e35780fba67787deeda14b0bdcf282a788328335a04fd21ac790bbaa1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter Notebook
  - Python
  - Python 3.10
  - SPIBACK
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# structured-table-serialization-csv-json

## Summary

Extracts reported numeric parameters from a scientific article (here, the fiducial Milky Way bar/spiral pattern speeds and contrast surface densities of Khalil et al. 2025) and serializes them as a validated CSV/JSON table via Python's pandas or csv module, so downstream agents can reuse the values without re-reading prose.

## When to use

Use when a task asks to reproduce a published parameter table — e.g. bar pattern speed 37 km/s/kpc, two-armed spiral 13.1 km/s/kpc (~25% contrast), three-armed 16.4 km/s/kpc (~9% contrast) — as a machine-readable CSV/JSON artifact.

## When NOT to use

- When the goal is to re-run the SPIBACK backward-integration model itself (spiback_minimalversion_run.ipynb) rather than tabulate its already-published fiducial parameters
- When the parameters already exist in a structured, machine-readable format (e.g. a released data table) — re-parsing prose would risk transcription error
- When values must be recomputed from Gaia DR3 velocity maps instead of quoted from the paper's reported fiducial model

## Inputs

- Published article text (A&A DOI 10.1051/0004-6361/202453077), abstract/results sections of Khalil et al. 2025
- Extracted finding passages reporting the fiducial model parameters (bar, two-armed spiral, three-armed spiral)

## Outputs

- fiducial_model_parameters.csv with columns: component, parameter, value, unit
- Optionally the same table serialized as JSON rows for programmatic reuse

## How to apply

Load the article text (A&A, DOI 10.1051/0004-6361/202453077) and locate the abstract/results passages describing the fiducial model. Parse the reported values: bar pattern speed of 37 km/s/kpc, two-armed spiral mode pattern speed of 13.1 km/s/kpc with ~25% local contrast surface density, and three-armed spiral mode pattern speed of 16.4 km/s/kpc with ~9% local contrast density. Build a pandas DataFrame (or csv.writer rows) with explicit columns: component (bar, two-armed spiral, three-armed spiral), parameter name (pattern speed, local contrast density), value, and unit (km/s/kpc or fraction). Write the table to fiducial_model_parameters.csv with index=False. Validate by string/numeric comparison of every output field against the abstract text — a field is correct only if it matches 37, 13.1, 16.4 km/s/kpc and the ~25%/~9% contrasts; handle the '~' prefix explicitly so it does not break float parsing.

## Related tools

- **Python 3.10** (Runs the parsing and serialization script; the workflow specifies tabulating parameters with pandas/Python's csv module)
- **Jupyter Notebook** (Environment in which the extraction-and-serialization workflow is executed and the output table checked)
- **SPIBACK** (Source of the fiducial model parameters being tabulated; its default settings correspond to the fiducial model described in Khalil et al.) — github.com/yrkhalil/SPIBACK

## Examples

```
import pandas as pd; pd.DataFrame(rows, columns=['component','parameter','value','unit']).to_csv('fiducial_model_parameters.csv', index=False)
```

## Evaluation signals

- Every numeric field in fiducial_model_parameters.csv matches the paper's reported values exactly: 37, 13.1, and 16.4 km/s/kpc
- Contrast fields parse correctly despite the '~' approximation prefix, yielding 0.25 (two-armed) and 0.09 (three-armed) as fractions or 25%/9%
- Schema check: the CSV has the four required columns (component, parameter, value, unit) with one row per component-parameter pair and no NaN or unparsed cells
- Round-trip check: reading the CSV back with pandas and comparing against strings extracted from the abstract passes for all rows
- Units are correctly assigned: km/s/kpc for the three pattern-speed rows, fraction for the two contrast rows

## Limitations

- The reported contrasts are approximations ('~25%', '~9%') from the abstract, so exact-precision validation is impossible; only the pattern speeds (37, 13.1, 16.4) are exact numbers
- Unicode superscript formatting in 'km s−1 kpc−1' must be normalized to km/s/kpc or parsing may fail
- The full parameter set lives in the paper and the fuller SPIBACK version is not yet released, so values beyond those quoted in the abstract cannot be tabulated from this source

## Evidence

- [abstract] This fiducial model has a bar with a pattern speed of 37 km s−1 kpc−1 and two spiral modes that are twoarmed and three-armed, respectively.: "This fiducial model has a bar with a pattern speed of 37 km s−1 kpc−1 and two spiral modes that are twoarmed and three-armed, respectively."
- [abstract] The two-armed spiral mode has a ~25% local contrast surface density and a low pattern speed of 13.1 km s−1 kpc−1, and matches the location of the Crux-Scutum, Local, and Outer arm segments.: "The two-armed spiral mode has a ~25% local contrast surface density and a low pattern speed of 13.1 km s−1 kpc−1"
- [abstract] The three-armed spiral mode has a ~9% local contrast density, a slightly higher pattern speed of 16.4 km s−1 kpc−1, and matches the location of the Carina-Sagittarius and Perseus arm segments.: "The three-armed spiral mode has a ~9% local contrast density, a slightly higher pattern speed of 16.4 km s−1 kpc−1"
- [readme] This is a Python3 notebook, which needs Python 3.10 or above.: "This is a Python3 notebook, which needs Python 3.10 or above."
- [intro] The parameters are set by default to be those of the "fiducial model", see Khalil et al for all the details: "The parameters are set by default to be those of the "fiducial model", see Khalil et al for all the details"
