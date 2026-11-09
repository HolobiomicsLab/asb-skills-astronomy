---
name: insitu-probability-contour-plotting
description: Use when a completed OSPREI ensemble run (nRuns > 1) has produced standardized
  output files and you need probability visualizations of FIDO in situ properties
  or ensemble correlations rather than raw data files.
when_to_use_negative:
- A single (non-ensemble) simulation was run — the processOSPREI routine makes an
  assortment of figures depending on whether a single simulation or a full ensemble
  was run, so probability contour plots from ensemble variation would not apply.
- You need quick figures while still testing an OSPREI configuration — the contour
  and correlation plots are the most computationally expensive and the README advises
  commenting them out for fast turnaround.
- You only need raw arrival-time or speed numbers rather than visualizations — the
  standardized output files already contain these directly.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: OSPREI
  role: Generates the standardized ensemble output data files that processOSPREI.py
    converts into figures; invoked with the same input text file
  repo: https://github.com/ckay314/OSPREI
- name: processOSPREI.py
  role: Post-processing routine that loads standardized outputs into a results object
    and automatically produces probability contour plots, correlation plots, profiles,
    histograms, and spatial maps
  repo: https://github.com/ckay314/OSPREI
- name: ForeCAT
  role: Supplies coronal positional profiles used for spatial maps and coronal trajectory
    ensemble figures in the results object
  repo: https://github.com/ckay314/OSPREI
- name: FIDO
  role: Produces the in situ profiles whose ensemble probabilities are rendered as
    contour plots
  repo: https://github.com/ckay314/OSPREI
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1029/2021sw002914
    title: 'OSPREI: A Coupled Approach to Modeling CME‐Driven Space Weather With Automatically
      Generated, User‐Friendly Outputs'
  source_content_hash: sha256:15b202398fedae95cdd803d424f37e1f741516baecf4824c1ced07088978fd42
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
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# insitu-probability-contour-plotting

## Summary

Generates in situ probability contour plots (and related ensemble correlation plots) from OSPREI ensemble output files using the processOSPREI.py post-processing routine, which loads standardized ForeCAT/ANTEATR/FIDO outputs into a results object and renders user-friendly figures for forecasters.

## When to use

Use when a completed OSPREI ensemble run (nRuns > 1) has produced standardized output files and you need probability visualizations of FIDO in situ properties or ensemble correlations rather than raw data files.

## When NOT to use

- A single (non-ensemble) simulation was run — the processOSPREI routine makes an assortment of figures depending on whether a single simulation or a full ensemble was run, so probability contour plots from ensemble variation would not apply.
- You need quick figures while still testing an OSPREI configuration — the contour and correlation plots are the most computationally expensive and the README advises commenting them out for fast turnaround.
- You only need raw arrival-time or speed numbers rather than visualizations — the standardized output files already contain these directly.

## Inputs

- Standardized output data files from a completed OSPREI ensemble run (ForeCAT, ANTEATR/PARADE, and FIDO outputs)
- OSPREI input text file (e.g., OSPtestcase.txt) with the required date, time, suffix parameters and nRuns > 1
- .ens ensemble file specifying varied parameters and their ranges

## Outputs

- In situ probability contour plots as image files
- Ensemble correlation plots
- Parameter-versus-distance profiles and histograms for ForeCAT, ANTEATR, and FIDO ensemble outputs
- Spatial maps and coronal trajectory ensemble figures from ForeCAT positional profiles
- A populated results object holding all simulation data for custom figures

## How to apply

Run the post-processor as 'python processOSPREI.py OSPtestcase.txt', passing the same input text file used for the OSPREI run; it loads the standardized output files into a results object before making figures. Among the generated figures, compute the in situ probability contour plots and ensemble correlation plots, which require more significant computation than the straightforward profiles-versus-distance and histogram figures for ForeCAT, ANTEATR, and FIDO outputs. If quick turnaround figures are needed during testing, comment these expensive contour/correlation sections out and run them later. Also produce spatial maps and coronal trajectory ensemble figures from the ForeCAT positional profiles held in the results object. For any visualization beyond the standardized set, build custom figures directly from the results object, which contains all simulation data in an easily accessible format.

## Related tools

- **OSPREI** (Generates the standardized ensemble output data files that processOSPREI.py converts into figures; invoked with the same input text file) — https://github.com/ckay314/OSPREI
- **processOSPREI.py** (Post-processing routine that loads standardized outputs into a results object and automatically produces probability contour plots, correlation plots, profiles, histograms, and spatial maps) — https://github.com/ckay314/OSPREI
- **ForeCAT** (Supplies coronal positional profiles used for spatial maps and coronal trajectory ensemble figures in the results object) — https://github.com/ckay314/OSPREI
- **FIDO** (Produces the in situ profiles whose ensemble probabilities are rendered as contour plots) — https://github.com/ckay314/OSPREI

## Examples

```
python processOSPREI.py OSPtestcase.txt
```

## Evaluation signals

- processOSPREI.py runs without error on the ensemble outputs using the same input text file as the OSPREI run
- Expected figure files — profiles, histograms, in situ probability contour plots — are produced as valid image files
- Figures correctly reflect whether a single simulation or a full ensemble was run (contours/correlations appear only for ensemble cases)
- The results object is populated and can be used to construct additional custom figures beyond the standardized set

## Limitations

- The in situ probability contour plots and ensemble correlation plots require more significant computation than other figures and slow turnaround during testing; the README recommends commenting them out when quick figures are needed
- processOSPREI.py automatically names output files based on the date and suffix, so runs for multiple configurations or CMEs on the same day need distinct suffix tags to avoid overwriting
- Only standardized figures are generated by default; anything beyond the standard set must be hand-built from the results object

## Evidence

- [readme] Post-processing routine converts standardized files to figures: "Systematic output files are generated, which can then be automatically processed into user-friendly figures."
- [readme] Invocation pattern matches the main OSPREI call: "python processOSPREI.py OSPtestcase.txt"
- [readme] Contour and correlation plots are the computationally expensive figures: "Others require more significant computation, particularly for the contour plots and ensemble correltation plots, and should potentially be commented out and not run if you want to make quick figures"
- [readme] Results object enables custom figures: "New figures can be created using the results object that contains all the simulation data in an easily accessible format."
- [intro] Standardized visualizations are a headline OSPREI feature: "Standardized visualizations are automatically generated, providing easily accessible, essential information for space weather forecasting."
