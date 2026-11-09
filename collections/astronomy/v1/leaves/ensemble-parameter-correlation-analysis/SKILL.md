---
name: ensemble-parameter-correlation-analysis
description: Use when you have standardized OSPREI ensemble output files (from a '.ens'
  configuration varying input ranges) and need to identify which varied inputs drive
  which quantifiable outputs, e.g. final latitude, longitude, tilt, arrival time,
  speed, or in situ properties.
when_to_use_negative:
- You only need quick-look figures (profiles versus distance or histograms) while
  testing — correlation plots are computationally expensive and should be commented
  out for fast turnaround
- You ran a single simulation rather than an ensemble (nRuns = 1), so there are no
  varied input ranges to correlate against outputs
- You need custom figures beyond the standardized set built from scratch rather than
  the standardized correlation panels
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: OSPREI
  role: Generates the ensemble output data files and varied-parameter structure ('.ens'
    ranges) that the correlation analysis is computed over
  repo: https://github.com/ckay314/OSPREI
- name: processOSPREI.py
  role: Loads standardized ensemble outputs into a results object and computes the
    ensemble correlation plots (contour and correlation figures)
  repo: https://github.com/ckay314/OSPREI
- name: ForeCAT
  role: Supplies coronal trajectory outputs (final latitude, longitude, tilt and their
    profiles versus distance) used as correlated ensemble outputs
  repo: https://github.com/ckay314/OSPREI
- name: ANTEATR
  role: Supplies interplanetary propagation outputs (arrival time, speed, size, shape)
    used as correlated ensemble outputs
- name: FIDO
  role: Supplies in situ profile outputs (magnetic field, velocity, Kp) used as correlated
    ensemble outputs
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1029/2021sw002914
    title: 'OSPREI: A Coupled Approach to Modeling CME‐Driven Space Weather With Automatically
      Generated, User‐Friendly Outputs'
  source_content_hash: sha256:850ee2892868039c794e0317e65807330d09a8505b940a4f22a843d2f0958777
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ckay314/OSPREI
  tools:
  - ANTEATR
  - FIDO
  - ForeCAT
  - OSPREI
  - processOSPREI.py
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# ensemble-parameter-correlation-analysis

## Summary

Quantifies which varied ensemble input parameters and quantifiable outputs of OSPREI ensemble runs are meaningfully linked by computing correlation coefficients and applying a 0.5 correlation-magnitude cutoff, then assembling the selected input-output pairs into per-CME correlation tables/plots as in Figure 9.

## When to use

Use when you have standardized OSPREI ensemble output files (from a '.ens' configuration varying input ranges) and need to identify which varied inputs drive which quantifiable outputs, e.g. final latitude, longitude, tilt, arrival time, speed, or in situ properties.

## When NOT to use

- You only need quick-look figures (profiles versus distance or histograms) while testing — correlation plots are computationally expensive and should be commented out for fast turnaround
- You ran a single simulation rather than an ensemble (nRuns = 1), so there are no varied input ranges to correlate against outputs
- You need custom figures beyond the standardized set built from scratch rather than the standardized correlation panels

## Inputs

- Standardized output data files from a completed OSPREI ensemble run (nRuns > 1)
- .ens ensemble configuration file specifying varied parameters and their ranges
- Results object built by processOSPREI.py from the ensemble outputs

## Outputs

- Ensemble correlation plots (scatter-plot panels of input-output pairs exceeding the 0.5 cutoff)
- Per-case correlation table of selected input-output pairs (e.g. 6 inputs x 11 outputs per CME)
- Correlation coefficients bounded in [-1, 1] for each retained pair

## How to apply

Load the standardized output files from the OSPREI ensemble runs into a results object using processOSPREI.py, reading the varied-parameter ranges defined in the '.ens' ensemble configuration files. For each CME case, compute the correlation coefficient between every varied ensemble input parameter and every quantifiable output. Apply a 0.5 correlation-magnitude cutoff to select which input-output pairs are retained for the ensemble correlation panels. Assemble a per-case correlation table (e.g. 6 varied inputs by 11 outputs exceeding the cutoff per CME) and render the scatter-plot panels via the ensemble correlation plotting routine in processOSPREI.py. Note these correlation plots are among the 'more significant computation' figures, so skip or comment them out when only quick-turnaround figures are needed during testing.

## Related tools

- **OSPREI** (Generates the ensemble output data files and varied-parameter structure ('.ens' ranges) that the correlation analysis is computed over) — https://github.com/ckay314/OSPREI
- **processOSPREI.py** (Loads standardized ensemble outputs into a results object and computes the ensemble correlation plots (contour and correlation figures)) — https://github.com/ckay314/OSPREI
- **ForeCAT** (Supplies coronal trajectory outputs (final latitude, longitude, tilt and their profiles versus distance) used as correlated ensemble outputs) — https://github.com/ckay314/OSPREI
- **ANTEATR** (Supplies interplanetary propagation outputs (arrival time, speed, size, shape) used as correlated ensemble outputs)
- **FIDO** (Supplies in situ profile outputs (magnetic field, velocity, Kp) used as correlated ensemble outputs)

## Examples

```
python processOSPREI.py OSPtestcase.txt
```

## Evaluation signals

- All tabulated correlation coefficients lie within [-1, 1]
- The correlation table reproduces the expected structure — e.g. 6 varied inputs and 11 outputs exceeding the 0.5 cutoff per CME (not necessarily the same parameters for each CME)
- CME 1 exhibits strong correlations of final latitude, longitude, and tilt with their initial values
- Every retained input-output pair has |correlation coefficient| > 0.5 (the applied cutoff)
- The ensemble correlation plot files are produced as valid image files without script error

## Limitations

- The correlation cutoff of 0.5 is a fixed threshold; pairs just below it are excluded even if physically meaningful
- Different CME cases need not share the same set of correlated parameters, so results are case-specific and not directly transferable
- Ensemble correlation plots are computationally expensive compared with profiles and histograms, hindering quick-turnaround testing
- Only two CME cases were considered in the study, so conclusions about parameter-output relationships are not yet established broadly

## Evidence

- [full_text] Apply the 0.5 correlation-magnitude cutoff to select which input-output pairs are included in the ensemble correlation panels.: [evidence span withheld — anchor status: not_found]
- [readme] README on correlation plots being expensive: "Others require more significant computation, particularly for the contour plots and ensemble correltation plots, and should potentially be commented out and not run if you want to make quick figures"
- [readme] README on results object enabling new figures: "New figures can be created using the results object that contains all the simulation data in an easily accessible format."
- [results] Article finding on Figure 9 structure: [evidence span withheld — anchor status: not_found]
- [intro] Article on ensemble uncertainty probing: "OSPREI includes ensemble modeling by varying each input parameter to probe any uncertainty in their values, yielding probabilities for all outputs."
