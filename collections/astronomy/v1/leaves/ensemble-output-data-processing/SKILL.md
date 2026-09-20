---
name: ensemble-output-data-processing
description: Use when OSPREI ensemble runs (nRuns > 1 with a '.ens' configuration
  file) have completed for one or more CMEs and the standardized output files must
  be converted into figures, histograms, contour plots, or correlation panels.
when_to_use_negative:
- The OSPREI simulation itself has not been run yet — processOSPREI.py operates only
  on standardized output files produced after running OSPREI.py
- Only a single (non-ensemble) simulation was run — some figures such as ensemble
  correlation panels do not apply
- Quick prototype figures are needed during testing — correlation and contour plots
  require significant computation and should be commented out
edam_operation: http://edamontology.org/operation_2409
tools:
- name: OSPREI
  role: Suite whose standardized ensemble output files are loaded and processed into
    figures and correlation panels via processOSPREI.py
  repo: https://github.com/ckay314/OSPREI
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1029/2021sw002914
    title: 'OSPREI: A Coupled Approach to Modeling CME‐Driven Space Weather With Automatically
      Generated, User‐Friendly Outputs'
  source_content_hash: sha256:4e3d79e31e7546ac76136916b9cbfefe3f3b49ed406566aac7406e21cba420da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ckay314/OSPREI
  tools:
  - OSPREI
  edam_operation: http://edamontology.org/operation_2409
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# ensemble-output-data-processing

## Summary

Loads and processes standardized OSPREI ensemble output files into a results object via processOSPREI.py, then computes input–output correlation tables (e.g., 0.5 correlation-magnitude cutoff yielding 6 inputs × 11 outputs per CME) that drive the ensemble correlation panels.

## When to use

Use when OSPREI ensemble runs (nRuns > 1 with a '.ens' configuration file) have completed for one or more CMEs and the standardized output files must be converted into figures, histograms, contour plots, or correlation panels.

## When NOT to use

- The OSPREI simulation itself has not been run yet — processOSPREI.py operates only on standardized output files produced after running OSPREI.py
- Only a single (non-ensemble) simulation was run — some figures such as ensemble correlation panels do not apply
- Quick prototype figures are needed during testing — correlation and contour plots require significant computation and should be commented out

## Inputs

- OSPREI input text file (e.g., OSPtestcase.txt) with nRuns set > 1
- Companion '.ens' ensemble configuration file specifying varied parameters and their ranges
- Standardized output files from OSPREI ensemble runs (e.g., 22 April and 09 May 2021 CMEs)

## Outputs

- Results object containing all simulation data in an easily accessible format
- Automatically generated figures: profiles versus distance, histograms, contour plots, and ensemble correlation panels
- Per-CME correlation table of input-output pairs exceeding the 0.5 cutoff (6 inputs × 11 outputs per CME for the studied cases)

## How to apply

Run processOSPREI.py with the same input text file used for OSPREI.py; it reads the standardized output files, builds a results object containing all simulation data, and generates the default figure set. The ensemble configuration (varied parameters and their ranges) is defined in a companion '.ens' file matched to the input file name, triggered by setting nRuns > 1. For ensemble correlation analysis, compute the correlation coefficient between every varied input parameter and every quantifiable output (e.g., final latitude, longitude, tilt, arrival time, speed, in situ properties) and apply a correlation-magnitude cutoff of 0.5 to select pairs for the ensemble correlation panels; in the 22 April and 09 May 2021 CME cases this retains 6 inputs and 11 outputs per CME, though not entirely the same parameters for each. Verify that all tabulated coefficients lie in [-1, 1] and that CME 1 shows strong correlations of final latitude, longitude, and tilt with their initial values. Note that correlation plots and contour plots are computationally heavier than simple profiles/histograms and may be commented out for quick figure generation.

## Related tools

- **OSPREI** (Suite whose standardized ensemble output files are loaded and processed into figures and correlation panels via processOSPREI.py) — https://github.com/ckay314/OSPREI

## Examples

```
python processOSPREI.py OSPtestcase.txt
```

## Evaluation signals

- All tabulated correlation coefficients lie in [-1, 1]
- Per-CME correlation table reproduces the reported structure: 6 varied inputs and 11 outputs exceeding the 0.5 correlation-magnitude cutoff for both CMEs
- CME 1 exhibits strong correlations of final latitude, longitude, and tilt with their initial values
- Results object is populated without error and default figure set (profiles, histograms) renders; '.ens' file was found because nRuns > 1

## Limitations

- Ensemble correlation plots and contour plots require more significant computation than simple profiles/histograms and may need to be commented out when generating quick figures during testing
- The retained parameters above the cutoff are not entirely the same for each CME, so per-CME correlation tables cannot be assumed identical
- Figure availability depends on whether a single simulation or a full ensemble was run

## Evidence

- [readme] After running OSPREI, the results can be automatically processed using `processOSPREI.py`. This takes the standardized output files, processes them into a results object, then makes an assortment of figures depending on whether a single simulation or a full ensemble was run.: "After running OSPREI, the results can be automatically processed using `processOSPREI.py`. This takes the standardized output files, processes them into a results object"
- [readme] Ensemble configuration file definition: "If it is set greater than 1 then the program looks for a second file with the same name as the input file but extension ".ens" that sets the parameters varied in the ensemble and their range."
- [results] Correlation cutoff and table structure: [evidence span withheld — anchor status: not_found]
- [readme] Computational cost caveat: "Others require more significant computation, particularly for the contour plots and ensemble correltation plots, and should potentially be commented out and not run if you want to make quick figures"
