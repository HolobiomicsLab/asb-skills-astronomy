---
name: cme-trajectory-sensitivity-interpretation
description: Use when OSPREI ensemble output files (from '.ens'-configured runs, e.g.,
  the 22 April and 09 May 2021 CMEs) need input-output correlation analysis to reveal
  which parameter uncertainties drive forecast outputs like arrival time, speed, and
  in situ properties.
when_to_use_negative:
- The run is a single simulation (nRuns = 1) with no ensemble spread — correlations
  across members are undefined
- The goal is a best-fit tuning of input parameters to match observations rather than
  uncertainty/sensitivity probing (the article explicitly avoids fine-tuning inputs)
- The analysis targets arrival-time error statistics against observational benchmarks
  (e.g., WSA-ENLIL + Cone comparisons) rather than internal ensemble input-output
  correlations
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: OSPREI
  role: Coupled Sun-to-Earth CME modeling suite whose ensemble runs produce the input-output
    data correlated in this skill; run via OSPREI.py with an '.ens' file for ensembles
  repo: https://github.com/ckay314/OSPREI
- name: processOSPREI.py
  role: Loads standardized OSPREI output files into a results object and generates
    the ensemble correlation scatter-plot figures
  repo: https://github.com/ckay314/OSPREI
- name: ForeCAT
  role: Produces final coronal latitude, longitude, and tilt outputs whose correlations
    with initial values validate the sensitivity analysis
  repo: https://github.com/ckay314/OSPREI
- name: ANTEATR (PARADE)
  role: Produces arrival time, speed, size, and shape outputs correlated against varied
    ensemble inputs
  repo: https://github.com/ckay314/OSPREI
- name: FIDO
  role: Produces synthetic in situ magnetic field and velocity profile outputs included
    among the correlated quantities
  repo: https://github.com/ckay314/OSPREI
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1029/2021sw002914
    title: 'OSPREI: A Coupled Approach to Modeling CME‐Driven Space Weather With Automatically
      Generated, User‐Friendly Outputs'
  source_content_hash: sha256:748c5ae76fc5091a1e2adb33b7efcc3f281d694f35ababa9b13aef5460692f58
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ckay314/OSPREI
  tools:
  - ANTEATR (PARADE)
  - FIDO
  - ForeCAT
  - OSPREI
  - processOSPREI.py
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cme-trajectory-sensitivity-interpretation

## Summary

Computes and interprets input-output correlations from OSPREI ensemble runs of Sun-to-Earth CME simulations (ForeCAT + ANTEATR/PARADE + FIDO), identifying which varied input parameters and quantifiable outputs exceed a 0.5 correlation-magnitude cutoff for each CME case.

## When to use

Use when OSPREI ensemble output files (from '.ens'-configured runs, e.g., the 22 April and 09 May 2021 CMEs) need input-output correlation analysis to reveal which parameter uncertainties drive forecast outputs like arrival time, speed, and in situ properties.

## When NOT to use

- The run is a single simulation (nRuns = 1) with no ensemble spread — correlations across members are undefined
- The goal is a best-fit tuning of input parameters to match observations rather than uncertainty/sensitivity probing (the article explicitly avoids fine-tuning inputs)
- The analysis targets arrival-time error statistics against observational benchmarks (e.g., WSA-ENLIL + Cone comparisons) rather than internal ensemble input-output correlations

## Inputs

- OSPREI standardized ensemble output files from runs configured via '.ens' ensemble files (nRuns > 1)
- Ensemble configuration '.ens' text files defining varied parameters and their ranges
- Varied input parameter values (e.g., CMElat, CMElon, CMEtilt, CMEvr, CMEAW, CMEM, SWv)
- Quantifiable output values (final latitude, longitude, tilt, arrival time, speed, in situ magnetic field and velocity profiles, Kp)

## Outputs

- Per-CME correlation coefficient table of input-output pairs exceeding the 0.5 magnitude cutoff
- Ensemble correlation scatter-plot panels (Figure 9-style) of selected input-output pairs
- Interpretation of which parameter uncertainties dominate output variability per CME

## How to apply

Load the standardized output files from the OSPREI ensemble runs into a results object using processOSPREI.py, reading the varied-parameter ranges defined in the '.ens' ensemble configuration files. For each CME, compute the correlation coefficient between every varied ensemble input parameter (e.g., CMElat, CMElon, CMEtilt, CMEvr, CMEAW) and every quantifiable output (final latitude, longitude, tilt, arrival time, speed, and in situ properties from FIDO). Apply a 0.5 correlation-magnitude cutoff to select which input-output pairs are shown in the Figure 9-style ensemble correlation scatter-plot panels. Assemble a per-CME correlation table; in the published analysis both CMEs yielded 6 varied inputs and 11 outputs exceeding the cutoff, though not entirely the same parameters for each CME. Validate that CME 1 exhibits strong correlations of final latitude, longitude, and tilt with their initial values, confirming the physical sensitivity of ForeCAT's coronal deflection and rotation.

## Related tools

- **OSPREI** (Coupled Sun-to-Earth CME modeling suite whose ensemble runs produce the input-output data correlated in this skill; run via OSPREI.py with an '.ens' file for ensembles) — https://github.com/ckay314/OSPREI
- **processOSPREI.py** (Loads standardized OSPREI output files into a results object and generates the ensemble correlation scatter-plot figures) — https://github.com/ckay314/OSPREI
- **ForeCAT** (Produces final coronal latitude, longitude, and tilt outputs whose correlations with initial values validate the sensitivity analysis) — https://github.com/ckay314/OSPREI
- **ANTEATR (PARADE)** (Produces arrival time, speed, size, and shape outputs correlated against varied ensemble inputs) — https://github.com/ckay314/OSPREI
- **FIDO** (Produces synthetic in situ magnetic field and velocity profile outputs included among the correlated quantities) — https://github.com/ckay314/OSPREI

## Examples

```
python processOSPREI.py OSPtestcase.txt
```

## Evaluation signals

- All tabulated correlation coefficients lie within [-1, 1]
- The correlation table reproduces the reported structure of 6 varied inputs and 11 outputs exceeding the 0.5 cutoff per CME (for the 22 April and 09 May 2021 cases)
- CME 1 shows strong correlations of final latitude, longitude, and tilt with their initial values
- The same cutoff (0.5) applied to both CMEs may select different parameter sets — differing panels between CMEs are expected, not an error
- Figures regenerate reproducibly via processOSPREI.py from the same standardized output files

## Limitations

- Only two CME cases (22 April and 09 May 2021) with relatively similar coronal origins but fairly different in situ properties were considered; more cases are needed to establish the model
- Exceeding-cutoff parameters are 'not entirely the same' for the two CMEs, so sensitivity conclusions are case-dependent
- Correlation panels and contour plots require significant computation in processOSPREI.py and may need to be commented out for quick figure generation during testing
- Parameter space was not optimized; inputs were not fine-tuned, so the best attainable results are unknown

## Evidence

- [other] 6 inputs and 11 outputs exceeding the cutoff per CME: [evidence span withheld — anchor status: not_found]
- [other] 0.5 correlation cutoff: [evidence span withheld — anchor status: not_found]
- [readme] ensemble correlation plots via processOSPREI: "Others require more significant computation, particularly for the contour plots and ensemble correltation plots, and should potentially be commented out"
- [intro] ensemble modeling rationale: "OSPREI includes ensemble modeling by varying each input parameter to probe any uncertainty in their values, yielding probabilities for all outputs."
