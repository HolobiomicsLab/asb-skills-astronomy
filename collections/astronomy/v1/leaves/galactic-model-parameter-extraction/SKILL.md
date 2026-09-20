---
name: galactic-model-parameter-extraction
description: Use when the task requires reproducing the reported fiducial parameter
  table (bar pattern speed 37 km/s/kpc; two-armed spiral at 13.1 km/s/kpc with ~25%
  local contrast; three-armed spiral at 16.4 km/s/kpc with ~9% local contrast) from
  published article text.
when_to_use_negative:
- When the goal is to re-fit or re-derive the parameters from Gaia DR3 kinematics
  via backward integration (use the SPIBACK notebook workflow instead of extraction
  from text)
- When parameters for a non-fiducial model are needed — the notebook defaults are
  set to the fiducial model, so custom parameter values must be set in spiback_minimalversion_run.ipynb
  rather than extracted from the abstract
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: Python 3.10
  role: Runs the extraction/tabulation script (notebook requires Python 3.10 or above)
- name: pandas
  role: Tabulate and serialize the extracted parameters as fiducial_model_parameters.csv
- name: Jupyter Notebook
  role: Environment where SPIBACK (whose defaults are the fiducial model parameters)
    is run for cross-checking the extracted values
  repo: https://github.com/yrkhalil/SPIBACK
- name: SPIBACK
  role: Jupyter Notebook whose default parameters correspond to the fiducial model
    described in Khalil et al., usable to confirm extracted values
  repo: https://github.com/yrkhalil/SPIBACK
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202453077
    title: A non-axisymmetric potential for the Milky Way disk
  source_content_hash: sha256:eda448ec6c404d409575eddf21cbabf06b5aade4b235d29a9aa269f2d5d70c31
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yrkhalil/SPIBACK
  tools:
  - Jupyter Notebook
  - Python
  - Python 3.10
  - SPIBACK
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# galactic-model-parameter-extraction

## Summary

Extracts and tabulates the fiducial Milky Way non-axisymmetric model parameters (bar and spiral pattern speeds and local contrast surface densities) from the Khalil et al. 2025 A&A paper, producing a structured CSV validated against the reported values.

## When to use

Use when the task requires reproducing the reported fiducial parameter table (bar pattern speed 37 km/s/kpc; two-armed spiral at 13.1 km/s/kpc with ~25% local contrast; three-armed spiral at 16.4 km/s/kpc with ~9% local contrast) from published article text.

## When NOT to use

- When the goal is to re-fit or re-derive the parameters from Gaia DR3 kinematics via backward integration (use the SPIBACK notebook workflow instead of extraction from text)
- When parameters for a non-fiducial model are needed — the notebook defaults are set to the fiducial model, so custom parameter values must be set in spiback_minimalversion_run.ipynb rather than extracted from the abstract

## Inputs

- Published article text (A&A DOI 10.1051/0004-6361/202453077), specifically the abstract/results passages on the fiducial model
- Parameter defaults from the SPIBACK notebook (spiback_minimalversion_run.ipynb), set to the fiducial model

## Outputs

- fiducial_model_parameters.csv with columns: component (bar / two-armed spiral / three-armed spiral), parameter name (pattern speed, local contrast density), value, unit (km/s/kpc or fraction)

## How to apply

Load the article text (A&A, DOI 10.1051/0004-6361/202453077) and locate the abstract/results passages describing the fiducial model. Parse the reported values: bar pattern speed of 37 km s−1 kpc−1, two-armed spiral mode at 13.1 km s−1 kpc−1 with ~25% local contrast surface density, and three-armed spiral mode at 16.4 km s−1 kpc−1 with ~9% local contrast density. Tabulate the parameters in Python (pandas or the csv module) with columns for component (bar, two-armed spiral, three-armed spiral), parameter name (pattern speed, local contrast density), value, and unit (km/s/kpc or fraction). Serialize as fiducial_model_parameters.csv. Validate by string/numeric comparison of every numeric field against the abstract text.

## Related tools

- **Python 3.10** (Runs the extraction/tabulation script (notebook requires Python 3.10 or above))
- **pandas** (Tabulate and serialize the extracted parameters as fiducial_model_parameters.csv)
- **Jupyter Notebook** (Environment where SPIBACK (whose defaults are the fiducial model parameters) is run for cross-checking the extracted values) — https://github.com/yrkhalil/SPIBACK
- **SPIBACK** (Jupyter Notebook whose default parameters correspond to the fiducial model described in Khalil et al., usable to confirm extracted values) — https://github.com/yrkhalil/SPIBACK

## Examples

```
import pandas as pd; df = pd.DataFrame({'component':['bar','two-armed spiral','three-armed spiral'],'pattern_speed_kms_kpc':[37,13.1,16.4],'local_contrast':[None,0.25,0.09]}); df.to_csv('fiducial_model_parameters.csv', index=False)
```

## Evaluation signals

- Every numeric field in fiducial_model_parameters.csv matches the paper's reported values exactly: 37, 13.1, 16.4 km/s/kpc; ~25% and ~9% contrast
- Pattern speed for the bar (37) is higher than both spiral modes (13.1 and 16.4), consistent with the paper's statement that the bar has a higher pattern speed than both spiral modes
- CSV schema contains exactly the columns: component, parameter name, value, unit
- Each component row carries the correct arm-to-segment association (two-armed: Crux-Scutum, Local, Outer; three-armed: Carina-Sagittarius, Perseus) as a consistency check on provenance

## Limitations

- Only a minimal version of the SPIBACK code is currently available for quick computations; a fuller version will be provided soon
- Values are approximations quoted with ~ (~25% and ~9% local contrast), so exact numeric comparison must tolerate the tilde qualifier
- The Discussion section text was absent from extraction; no discussion content supports cross-validation

## Evidence

- [abstract] bar and spiral pattern speeds: "This fiducial model has a bar with a pattern speed of 37 km s−1 kpc−1 and two spiral modes that are twoarmed and three-armed, respectively."
- [abstract] two-armed mode parameters: "The two-armed spiral mode has a ~25% local contrast surface density and a low pattern speed of 13.1 km s−1 kpc−1"
- [abstract] three-armed mode parameters: "The three-armed spiral mode has a ~9% local contrast density, a slightly higher pattern speed of 16.4 km s−1 kpc−1"
- [intro] notebook defaults: "The parameters are set by default to be those of the "fiducial model", see Khalil et al for all the details"
- [readme] minimal code caveat: "Currently only a minimal version of the code is available for quick computations. A fuller version will be provided soon."
