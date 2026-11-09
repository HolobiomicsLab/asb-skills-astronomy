---
name: non-axisymmetric-potential-parameter-tabulation
description: Use when reproducing or validating the reported fiducial parameter table
  of the Khalil et al. non-axisymmetric Milky Way model (bar and spiral pattern speeds
  and contrast densities) from the A&A article text.
when_to_use_negative:
- The article text or fiducial model passages are unavailable, so reported values
  (37, 13.1, 16.4 km/s/kpc; ~25%, ~9%) cannot be verified against the source — do
  not fabricate the table
- The goal is to re-fit or re-derive bar/spiral parameters from Gaia DR3 kinematic
  data via the backward integration method (use the SPIBACK notebook workflow instead),
  rather than to tabulate the already-reported fiducial values
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: Python
  role: Tabulate the parsed fiducial parameters (component, parameter name, value,
    unit) and serialize the table as a CSV
- name: Jupyter Notebook
  role: Interactive environment in which the parsing, tabulation, and CSV serialization
    steps are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202453077
    title: A non-axisymmetric potential for the Milky Way disk
  source_content_hash: sha256:e9e2e2103aa54879e9e6128af7eeec6373ad56523967d61a962eb140e999f1a2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter Notebook
  - Python
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# non-axisymmetric-potential-parameter-tabulation

## Summary

Tabulates the reported parameters of the fiducial non-axisymmetric Milky Way disk model of Khalil et al. (A&A, DOI 10.1051/0004-6361/202453077): bar pattern speed, and two-/three-armed spiral mode pattern speeds and local contrast surface densities, into a structured CSV for reuse.

## When to use

Use when reproducing or validating the reported fiducial parameter table of the Khalil et al. non-axisymmetric Milky Way model (bar and spiral pattern speeds and contrast densities) from the A&A article text.

## When NOT to use

- The article text or fiducial model passages are unavailable, so reported values (37, 13.1, 16.4 km/s/kpc; ~25%, ~9%) cannot be verified against the source — do not fabricate the table
- The goal is to re-fit or re-derive bar/spiral parameters from Gaia DR3 kinematic data via the backward integration method (use the SPIBACK notebook workflow instead), rather than to tabulate the already-reported fiducial values

## Inputs

- Published article text from A&A (DOI 10.1051/0004-6361/202453077), specifically the abstract/results passages describing the fiducial model
- Python environment with pandas or the csv standard-library module

## Outputs

- fiducial_model_parameters.csv: a table with columns component (bar, two-armed spiral, three-armed spiral), parameter name (pattern speed, local contrast density), value, and unit (km/s/kpc or fraction)

## How to apply

Load the published A&A article (DOI 10.1051/0004-6361/202453077) and locate the abstract/results passages describing the fiducial model, which has a bar with pattern speed 37 km s−1 kpc−1 plus two spiral modes. Parse the reported values: two-armed spiral mode pattern speed of 13.1 km/s/kpc with ~25% local contrast surface density, and three-armed spiral mode pattern speed of 16.4 km/s/kpc with ~9% local contrast density. Build a table in Python with columns for component (bar, two-armed spiral, three-armed spiral), parameter name (pattern speed, local contrast density), value, and unit (km/s/kpc or fraction), then serialize it as fiducial_model_parameters.csv using pandas or Python's csv module. Validate by string/numeric comparison of every numeric field in the output against the paper's reported values (37, 13.1, 16.4 km/s/kpc; ~25% and ~9% contrast).

## Related tools

- **Python** (Tabulate the parsed fiducial parameters (component, parameter name, value, unit) and serialize the table as a CSV)
- **Jupyter Notebook** (Interactive environment in which the parsing, tabulation, and CSV serialization steps are executed)

## Examples

```
import pandas as pd; pd.DataFrame({'component':['bar','two-armed spiral','three-armed spiral'],'pattern_speed_km_s_kpc':[37,13.1,16.4],'contrast':[None,0.25,0.09]}).to_csv('fiducial_model_parameters.csv', index=False)
```

## Evaluation signals

- Every numeric field in fiducial_model_parameters.csv matches the paper's reported values exactly: 37, 13.1, and 16.4 km/s/kpc for pattern speeds and ~25% and ~9% for local contrast densities, checked by string/numeric comparison against the abstract text
- The CSV contains rows for all three components (bar, two-armed spiral, three-armed spiral) with correct parameter names and units (km/s/kpc or fraction)
- The bar pattern speed (37 km/s/kpc) is higher than both spiral mode pattern speeds (13.1 and 16.4 km/s/kpc), consistent with the paper's finding that the bar has recently disconnected from the two spiral arms
- The two-armed mode (13.1 km/s/kpc) has a lower pattern speed than the three-armed mode (16.4 km/s/kpc), as reported

## Limitations

- Only the values explicitly reported in the abstract/results passages can be tabulated; contrast values are approximate (~25% and ~9%) rather than exact figures
- Currently only a minimal version of the SPIBACK code is available for quick computations; a fuller version will be provided soon, so this skill tabulates reported values rather than regenerating them
- Values are specific to the fiducial model of Khalil et al.; other bar/spiral parameter configurations would require the SPIBACK notebook with modified parameters

## Evidence

- [abstract] Fiducial Galactic bar model with pattern speed of 37 km/s/kpc and two spiral modes (two-armed and three-armed) adjusted to Gaia DR3 stellar kinematics: "This fiducial model has a bar with a pattern speed of 37 km s−1 kpc−1 and two spiral modes that are twoarmed and three-armed, respectively."
- [abstract] Two-armed spiral mode parameters (pattern speed and contrast): "The two-armed spiral mode has a ~25% local contrast surface density and a low pattern speed of 13.1 km s−1 kpc−1"
- [abstract] Three-armed spiral mode parameters (pattern speed and contrast): "The three-armed spiral mode has a ~9% local contrast density, a slightly higher pattern speed of 16.4 km s−1 kpc−1"
- [abstract] Bar disconnected from spiral arms because of higher pattern speed: "The Galactic bar, with a higher pattern speed than both spiral modes, has recently disconnected from those two arms."
- [readme] Default parameters match the fiducial model: "The parameters are set by default to be those of the "fiducial model", see Khalil et al for all the details: https://arxiv.org/abs/2411.12800"
