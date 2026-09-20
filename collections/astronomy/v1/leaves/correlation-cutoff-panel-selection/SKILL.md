---
name: correlation-cutoff-panel-selection
description: Use when analyzing OSPREI ensemble runs (e.g., the 22 April and 09 May
  2021 CMEs) to identify which varied input parameters correlate with which quantifiable
  outputs above a 0.5 cutoff.
when_to_use_negative:
- Single deterministic OSPREI simulations (nRuns = 1) — there are no ensemble members
  to correlate across; use profile/histogram figures instead
- Quick figure generation during testing — the README notes the correlation plots
  (like contour plots) require significant computation and can be commented out
edam_operation: http://edamontology.org/operation_0337
tools:
- name: OSPREI
  role: Ensemble simulation suite (ForeCAT + ANTEATR + FIDO) whose varied inputs and
    quantifiable outputs are correlated; run via OSPREI.py with an .ens ensemble file
  repo: https://github.com/ckay314/OSPREI
- name: processOSPREI.py
  role: Processes standardized ensemble output files into a results object and generates
    the ensemble correlation scatter-plot panels
  repo: https://github.com/ckay314/OSPREI
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1029/2021sw002914
    title: 'OSPREI: A Coupled Approach to Modeling CME‐Driven Space Weather With Automatically
      Generated, User‐Friendly Outputs'
  source_content_hash: sha256:f66af875dbc582bc851dc8759903494311339d5f8a28954802e9e8736ec0f35b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ckay314/OSPREI
  tools:
  - OSPREI
  - processOSPREI.py
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# correlation-cutoff-panel-selection

## Summary

Computes input-output correlation coefficients across OSPREI ensemble members and selects pairs exceeding a 0.5 correlation-magnitude cutoff to assemble the ensemble correlation scatter-plot panels (Figure 9).

## When to use

Use when analyzing OSPREI ensemble runs (e.g., the 22 April and 09 May 2021 CMEs) to identify which varied input parameters correlate with which quantifiable outputs above a 0.5 cutoff.

## When NOT to use

- Single deterministic OSPREI simulations (nRuns = 1) — there are no ensemble members to correlate across; use profile/histogram figures instead
- Quick figure generation during testing — the README notes the correlation plots (like contour plots) require significant computation and can be commented out

## Inputs

- Standardized output files from OSPREI ensemble runs (loaded into a results object via processOSPREI.py)
- '.ens' ensemble configuration files specifying which input parameters are varied and their ranges (e.g., CMElat, CMElon, CMEtilt, CMEvr, CMEAW, SWv)
- Ensemble run size set via nRuns > 1 in the OSPREI input text file

## Outputs

- Per-CME correlation table of input-output pairs exceeding the 0.5 correlation-magnitude cutoff (6 inputs x 11 outputs per CME for the two case studies)
- Ensemble correlation scatter-plot panels (Figure 9), one panel per selected input-output pair showing member scatter

## How to apply

Load the standardized output files from the OSPREI ensemble runs into a results object using processOSPREI.py, reading the varied-parameter ranges defined in the '.ens' ensemble configuration files (the '.ens' file is created when nRuns > 1). For each CME, compute the correlation coefficient between every varied ensemble input parameter and every quantifiable output (e.g., final latitude, longitude, tilt, arrival time, speed, in situ properties). Apply the 0.5 correlation-magnitude cutoff to select which input-output pairs are included in the ensemble correlation scatter-plot panels. Assemble a per-CME correlation table; for both case-study CMEs this yields 6 varied inputs by 11 outputs exceeding the cutoff, though not entirely the same parameters for each CME.

## Related tools

- **OSPREI** (Ensemble simulation suite (ForeCAT + ANTEATR + FIDO) whose varied inputs and quantifiable outputs are correlated; run via OSPREI.py with an .ens ensemble file) — https://github.com/ckay314/OSPREI
- **processOSPREI.py** (Processes standardized ensemble output files into a results object and generates the ensemble correlation scatter-plot panels) — https://github.com/ckay314/OSPREI

## Examples

```
python processOSPREI.py OSPtestcase.txt
```

## Evaluation signals

- All tabulated correlation coefficients lie in [-1, 1]
- Per-CME correlation table reproduces the reported structure of 6 varied inputs and 11 outputs exceeding the 0.5 cutoff for each CME (22 April and 09 May 2021)
- CME 1 exhibits strong correlations of final latitude, longitude, and tilt with their initial values
- Every included pair has |correlation coefficient| >= 0.5; excluded pairs fall below the cutoff
- Panels render as scatter plots of ensemble members for each selected input-output pair

## Limitations

- The selected parameters are not entirely the same for the two CMEs, so the cutoff-selected input-output sets are case-dependent
- Correlation plots require significant computation (like contour plots) and may need to be skipped when generating quick test figures
- Only two CME cases were considered, so the generality of the correlation structure is unestablished; the parameter space was not optimized

## Evidence

- [abstract] OSPREI includes ensemble modeling by varying each input parameter to probe any uncertainty in their values, yielding probabilities for all outputs.: "OSPREI includes ensemble modeling by varying each input parameter to probe any uncertainty in their values, yielding probabilities for all outputs."
- [readme] Others require more significant computation, particularly for the contour plots and ensemble correltation plots.: "Others require more significant computation, particularly for the contour plots and ensemble correltation plots, and should potentially be commented out and not run"
- [readme] If it is set greater than 1 then the program looks for a second file with the same name as the input file but extension ".ens": "If it is set greater than 1 then the program looks for a second file with the same name as the input file but extension ".ens" that sets the parameters varied in the ensemble and their range."
