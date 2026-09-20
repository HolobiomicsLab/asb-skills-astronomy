---
name: cme-trajectory-profile-visualization
description: Use when OSPREI (or a ForeCAT/ANTEATR/FIDO ensemble or single run) has
  produced standardized output data files and you need automated, forecast-ready figures
  of CME trajectories, parameter profiles, histograms, and ensemble statistics.
when_to_use_negative:
- You only need the underlying numeric data — the standardized output files and results
  object can be used directly without generating figures
- Quick turnaround during testing — contour plots and ensemble correlation plots are
  computationally expensive and should be skipped, not waited on
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: OSPREI
  role: Parent suite whose standardized output files are the input to the visualization
    routine; processOSPREI.py ships with it
  repo: https://github.com/ckay314/OSPREI
- name: ForeCAT
  role: Its positional profiles (latitude, longitude, tilt versus distance) feed the
    spatial maps and coronal trajectory ensemble figures
  repo: https://github.com/ckay314/OSPREI
- name: ANTEATR
  role: Its propagation/expansion outputs are rendered as profiles versus distance
    and histograms
  repo: https://github.com/ckay314/OSPREI
- name: FIDO
  role: Its in situ profiles feed probability contour plots and in situ figure generation
  repo: https://github.com/ckay314/OSPREI
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1029/2021sw002914
    title: 'OSPREI: A Coupled Approach to Modeling CME‐Driven Space Weather With Automatically
      Generated, User‐Friendly Outputs'
  source_content_hash: sha256:89ad4a7e04e7d59852cbeaf6a2866d2aae70db36a8391f936fc7ce587dda463f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ckay314/OSPREI
  tools:
  - ANTEATR
  - FIDO
  - ForeCAT
  - OSPREI
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# cme-trajectory-profile-visualization

## Summary

OSPREI's post-processing routine (processOSPREI.py) converts the standardized output data files from ForeCAT, ANTEATR, and FIDO runs into user-friendly figures — profiles of parameters versus distance, histograms, in situ probability contour plots, ensemble correlation plots, and spatial/coronal trajectory maps — via a results object holding all simulation data.

## When to use

Use when OSPREI (or a ForeCAT/ANTEATR/FIDO ensemble or single run) has produced standardized output data files and you need automated, forecast-ready figures of CME trajectories, parameter profiles, histograms, and ensemble statistics.

## When NOT to use

- You only need the underlying numeric data — the standardized output files and results object can be used directly without generating figures
- Quick turnaround during testing — contour plots and ensemble correlation plots are computationally expensive and should be skipped, not waited on

## Inputs

- Standardized output data files from a completed OSPREI run (ForeCAT, ANTEATR, FIDO), single-case or ensemble
- The OSPREI text input file used for the simulation (e.g. OSPtestcase.txt), passed on the command line so output folders/suffixes match

## Outputs

- Profile figures of parameters versus distance for ForeCAT, ANTEATR, and FIDO
- Histograms of final ensemble values (e.g. final latitude, longitude, tilt)
- In situ probability contour plots and ensemble correlation plots (ensemble cases)
- Spatial maps and coronal trajectory ensemble figures from ForeCAT positional profiles
- A results object holding all simulation data for custom figure construction

## How to apply

Invoke the routine by passing the same OSPREI text input file used for the simulation, e.g. 'python processOSPREI.py OSPtestcase.txt'; it loads the standardized output files into a results object. Generate the straightforward figures first: ForeCAT, ANTEATR, and FIDO profiles of latitude, longitude, tilt, speed, and size versus distance, plus final-value histograms. Compute the more expensive figures — in situ probability contour plots and ensemble correlation plots — only for full ensemble cases, and comment them out if quick turnaround is needed during testing. Produce spatial maps and coronal trajectory ensemble figures from the ForeCAT positional profiles in the results object, and use the results object to construct any additional custom figures. Validate that the script exits without error and the expected figure files (profiles, histograms, contour plots) exist as valid image files.

## Related tools

- **OSPREI** (Parent suite whose standardized output files are the input to the visualization routine; processOSPREI.py ships with it) — https://github.com/ckay314/OSPREI
- **ForeCAT** (Its positional profiles (latitude, longitude, tilt versus distance) feed the spatial maps and coronal trajectory ensemble figures) — https://github.com/ckay314/OSPREI
- **ANTEATR** (Its propagation/expansion outputs are rendered as profiles versus distance and histograms) — https://github.com/ckay314/OSPREI
- **FIDO** (Its in situ profiles feed probability contour plots and in situ figure generation) — https://github.com/ckay314/OSPREI

## Examples

```
python processOSPREI.py OSPtestcase.txt
```

## Evaluation signals

- Script runs without error on the ensemble outputs (e.g. 'python processOSPREI.py OSPtestcase.txt' completes)
- Expected figure files (profiles of values versus distance, histograms, contour plots) are produced as valid image files
- Figure file names/suffixes match the input file's date and suffix naming so OSPREI does not overwrite results from other configurations
- Figures reflect whether a single simulation or a full ensemble was run (single-case profiles vs. ensemble histograms/contours)
- Custom figures built on the results object correctly access the standardized simulation data it holds

## Limitations

- Contour plots and ensemble correlation plots require more significant computation and may need to be commented out for quick testing turnaround
- Figure generation depends on the standardized output file naming/folder conventions generated by OSPREI (date-based grouping with the suffix tag)
- Results beyond the standardized figure set require the user to build custom figures from the results object

## Evidence

- [readme] After running OSPREI, the results can be automatically processed using `processOSPREI.py`.: "After running OSPREI, the results can be automatically processed using `processOSPREI.py`."
- [intro] Standardized visualizations are automatically generated, providing easily accessible, essential information for space weather forecasting.: "Standardized visualizations are automatically generated, providing easily accessible, essential information for space weather forecasting."
- [readme] Some of the figures are very straightforward, such as profiles of values versus distance or histograms.: "Some of the figures are very straightforward, such as profiles of values versus distance or histograms."
- [readme] Contour plots and ensemble correlation plots are computationally expensive and should be skipped for quick testing: "Others require more significant computation, particularly for the contour plots and ensemble correltation plots, and should potentially be commented"
- [readme] New figures can be created using the results object that contains all the simulation data in an easily accessible format.: "New figures can be created using the results object that contains all the simulation data in an easily accessible format."
