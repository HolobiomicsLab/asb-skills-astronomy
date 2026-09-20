---
name: space-weather-forecast-figure-standardization
description: Use when you have completed an OSPREI run (single simulation or ensemble)
  and need to convert its systematic standardized output data files into user-friendly
  figures for space weather forecasting or further custom plotting.
when_to_use_negative:
- You need the physical simulation results themselves (arrival time, in situ profiles)
  — run OSPREI.py first; processOSPREI.py only post-processes existing output files.
- You need the PFSS magnetic background setup (magnetogram harmonics, PFSS.py pickles)
  — that is a prerequisite of the simulation, not of figure generation.
- 'Quick-iteration figure testing with expensive plots: skip/comment out the contour
  and ensemble correlation plots rather than waiting for full computation.'
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: OSPREI
  role: Generates the standardized output data files that processOSPREI.py consumes;
    the post-processor is part of the OSPREI suite
  repo: https://github.com/ckay314/OSPREI
- name: processOSPREI.py
  role: Post-processing routine that converts standardized output files into a results
    object and automatically generates the user-friendly figures
  repo: https://github.com/ckay314/OSPREI
- name: ForeCAT
  role: Its positional profiles (coronal deflection and rotation) feed the spatial
    maps and coronal trajectory ensemble figures
  repo: https://github.com/ckay314/OSPREI
- name: ANTEATR
  role: Its ensemble outputs (propagation, expansion, deformation) feed the profiles
    and histograms
  repo: https://github.com/ckay314/OSPREI
- name: FIDO
  role: Its in situ ensemble outputs feed the profiles, histograms, and in situ probability
    contour plots
  repo: https://github.com/ckay314/OSPREI
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1029/2021sw002914
    title: 'OSPREI: A Coupled Approach to Modeling CME‐Driven Space Weather With Automatically
      Generated, User‐Friendly Outputs'
  source_content_hash: sha256:f0583b5679b50d0ecaa126906d9b5b8d65a9b4ecc072c5e28e141f29456d2e1a
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
yes_rate: 0.8
n_claims: 10
---

# space-weather-forecast-figure-standardization

## Summary

Converts the standardized output data files from an OSPREI (ForeCAT + ANTEATR + FIDO) Sun-to-Earth CME run into user-friendly forecaster figures via the post-processing routine processOSPREI.py, which builds a results object and generates profiles, histograms, probability contour plots, ensemble correlation plots, and spatial/coronal trajectory maps.

## When to use

Use when you have completed an OSPREI run (single simulation or ensemble) and need to convert its systematic standardized output data files into user-friendly figures for space weather forecasting or further custom plotting.

## When NOT to use

- You need the physical simulation results themselves (arrival time, in situ profiles) — run OSPREI.py first; processOSPREI.py only post-processes existing output files.
- You need the PFSS magnetic background setup (magnetogram harmonics, PFSS.py pickles) — that is a prerequisite of the simulation, not of figure generation.
- Quick-iteration figure testing with expensive plots: skip/comment out the contour and ensemble correlation plots rather than waiting for full computation.

## Inputs

- Standardized output data files from a completed OSPREI run (ForeCAT, ANTEATR, FIDO outputs)
- OSPREI input text file (e.g. OSPtestcase.txt) whose name/filename suffix identifies the case files
- Flag distinguishing single-simulation vs full-ensemble runs (nRuns > 1)

## Outputs

- Parameter profiles versus distance figures
- Histograms for ForeCAT, ANTEATR, and FIDO ensemble outputs
- In situ probability contour plots
- Ensemble correlation plots
- Spatial maps and coronal trajectory ensemble figures from ForeCAT positional profiles
- Results object (Python object) containing all simulation data for custom figure creation

## How to apply

Invoke the post-processor as a separate routine after the main simulation: 'python processOSPREI.py OSPtestcase.txt', passing the same input text file used for the OSPREI.py run. The script loads the standardized output files into a results object that holds all simulation data in an easily accessible format, then generates the straightforward figures first: parameter profiles versus distance and histograms for the ForeCAT, ANTEATR, and FIDO ensemble outputs. The more expensive figures — in situ probability contour plots and ensemble correlation plots — should be commented out when quick turnaround figures are needed during testing. Spatial maps and coronal trajectory ensemble figures are produced from the ForeCAT positional profiles; any additional custom figures can be built directly from the results object.

## Related tools

- **OSPREI** (Generates the standardized output data files that processOSPREI.py consumes; the post-processor is part of the OSPREI suite) — https://github.com/ckay314/OSPREI
- **processOSPREI.py** (Post-processing routine that converts standardized output files into a results object and automatically generates the user-friendly figures) — https://github.com/ckay314/OSPREI
- **ForeCAT** (Its positional profiles (coronal deflection and rotation) feed the spatial maps and coronal trajectory ensemble figures) — https://github.com/ckay314/OSPREI
- **ANTEATR** (Its ensemble outputs (propagation, expansion, deformation) feed the profiles and histograms) — https://github.com/ckay314/OSPREI
- **FIDO** (Its in situ ensemble outputs feed the profiles, histograms, and in situ probability contour plots) — https://github.com/ckay314/OSPREI

## Examples

```
python processOSPREI.py OSPtestcase.txt
```

## Evaluation signals

- processOSPREI.py runs without error on the ensemble outputs
- Expected figure files (profiles, histograms, contour plots) are produced as valid image files
- Figures reflect the correct run type: single-simulation vs ensemble figure set is chosen automatically
- Results object loads with all simulation data accessible for constructing additional custom figures

## Limitations

- Contour plots and ensemble correlation plots require significant computation, especially for ensembles, and may need to be commented out for quick turnaround during testing
- The routine is separate from the main simulation; it cannot be run before OSPREI.py has produced its systematic output files
- Only the standardized figure set is automatic; anything beyond it requires writing custom figures from the results object

## Evidence

- [readme] processOSPREI invoked as a separate routine: "Systematic output files are generated, which can then be automatically processed into user-friendly figures.
```
python processOSPREI.py OSPtestcase.txt
```"
- [readme] results object and figure cost decision point: "This takes the standardized output files, processes them into a results object, then makes an assortment of figures depending on whether a single simulation or a full ensemble was run."
- [readme] expensive figures caveat: "Others require more significant computation, particularly for the contour plots and ensemble correltation plots, and should potentially be commented out and not run if you want to make quick figures"
- [readme] custom figures from results object: "New figures can be created using the results object that contains all the simulation data in an easily accessible format."
- [intro] automatic standardized visualizations for forecasting: "Standardized visualizations are automatically generated, providing easily accessible, essential information for space weather forecasting."
- [discussion] spatial maps from ForeCAT positional profiles: "We have presented the new CME forecasting suite OSPREI, which represents the combination of previously established models and includes standardized, user-friendly visualizations."
