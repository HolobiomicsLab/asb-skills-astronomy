---
name: latitude-longitude-contour-panel-visualization
description: Use when processing standardized OSPREI ensemble output (nRuns > 1 with
  a .ens file) into per-CME latitude–longitude contour panels showing spatial maps
  of impact probability and mean CME/space-weather properties at a satellite.
when_to_use_negative:
- 'Only a single simulation was run (nRuns = 1): histograms and profiles are produced,
  but ensemble-averaged contour panels over members are not meaningful'
- You want quick figures during testing — the README advises the contour and ensemble
  correlation plots 'should potentially be commented out and not run' due to significant
  computation
- Input is raw ForeCAT/ANTEATR/FIDO per-component output that has not yet been standardized
  by a full OSPREI run and processed by processOSPREI.py
edam_operation: http://edamontology.org/operation_0337
tools:
- name: OSPREI
  role: Suite whose standardized ensemble output files feed processOSPREI.py; processOSPREI.py
    renders the contour panels from the results object
  repo: https://github.com/ckay314/OSPREI
- name: processOSPREI.py
  role: Reads standardized outputs into a results object and generates the matplotlib-based
    latitude–longitude contour-panel figures
  repo: https://github.com/ckay314/OSPREI
- name: ForeCAT
  role: Provides each ensemble member's final latitude, longitude, and tilt used to
    project the CME footprint onto the lat–lon grid
  repo: https://github.com/ckay314/OSPREI
- name: ANTEATR
  role: Provides the evolved CME size, shape, and internal properties at impact that
    populate the averaged contour panels
  repo: https://github.com/ckay314/OSPREI
- name: FIDO
  role: Supplies in situ quantities (Bz, v_r, Kp) mapped across the lat–lon footprint
    at the synthetic satellite
  repo: https://github.com/ckay314/OSPREI
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1029/2021sw002914
    title: 'OSPREI: A Coupled Approach to Modeling CME‐Driven Space Weather With Automatically
      Generated, User‐Friendly Outputs'
  source_content_hash: sha256:ad92258c7db83afe2918a6982838114a557353cca6f52c2cf539e7ef9143da4f
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# latitude-longitude-contour-panel-visualization

## Summary

Generates ensemble-averaged latitude–longitude contour-panel figures from OSPREI output, projecting each ensemble member's CME footprint (flux rope plus optional sheath) at impact onto a lat–lon grid and accumulating per-bin statistics such as impact percentage, Bz, v_r, Kp, density, and log T.

## When to use

Use when processing standardized OSPREI ensemble output (nRuns > 1 with a .ens file) into per-CME latitude–longitude contour panels showing spatial maps of impact probability and mean CME/space-weather properties at a satellite.

## When NOT to use

- Only a single simulation was run (nRuns = 1): histograms and profiles are produced, but ensemble-averaged contour panels over members are not meaningful
- You want quick figures during testing — the README advises the contour and ensemble correlation plots 'should potentially be commented out and not run' due to significant computation
- Input is raw ForeCAT/ANTEATR/FIDO per-component output that has not yet been standardized by a full OSPREI run and processed by processOSPREI.py

## Inputs

- Standardized OSPREI output files from a completed ensemble run (nRuns > 1)
- .ens ensemble definition file setting varied parameters and their ranges (same base name as the input text file)
- OSP input text file with REQUIRED date (YYYYMMDD), time (HH:MM), and suffix parameters

## Outputs

- Seven-panel latitude–longitude contour figure per CME: impact percentage, mean duration, Bz, v_r, Kp, density, log T
- Figure files saved under the date- and suffix-based standardized naming convention
- Results object exposing all ensemble simulation data for constructing new custom figures

## How to apply

Load the standardized OSPREI output files with processOSPREI.py, which builds a results object holding every ensemble member's CME geometry (final position, orientation, size, shape from ForeCAT/ANTEATR-PARADE) and internal properties at impact. For each of the nRuns members defined by the .ens ensemble file, project the CME footprint at impact — flux rope plus optional sheath — onto a latitude–longitude grid using the member's final ForeCAT latitude, longitude, and tilt. Accumulate per-bin ensemble statistics (impact percentage, mean duration, Bz, v_r, Kp, density, log T) across all members. Render the seven averaged-property contour panels per CME using the OSPREI standardized matplotlib-based figure routines, and save the files with the date- and suffix-based naming convention (suffix is REQUIRED in the input file to avoid overwriting other runs). Note processOSPREI.py warns these contour plots are among the most computationally significant figures and can be commented out for quick test runs.

## Related tools

- **OSPREI** (Suite whose standardized ensemble output files feed processOSPREI.py; processOSPREI.py renders the contour panels from the results object) — https://github.com/ckay314/OSPREI
- **processOSPREI.py** (Reads standardized outputs into a results object and generates the matplotlib-based latitude–longitude contour-panel figures) — https://github.com/ckay314/OSPREI
- **ForeCAT** (Provides each ensemble member's final latitude, longitude, and tilt used to project the CME footprint onto the lat–lon grid) — https://github.com/ckay314/OSPREI
- **ANTEATR** (Provides the evolved CME size, shape, and internal properties at impact that populate the averaged contour panels) — https://github.com/ckay314/OSPREI
- **FIDO** (Supplies in situ quantities (Bz, v_r, Kp) mapped across the lat–lon footprint at the synthetic satellite) — https://github.com/ckay314/OSPREI

## Examples

```
python processOSPREI.py OSPtestcase.txt
```

## Evaluation signals

- Each map figure file exists on disk after processOSPREI.py completes, named per the date- and suffix-based convention
- All impact-percentage values in the contour maps fall within [0, 100]%
- CME 1 exhibits a rounded-rectangle nonzero-impact region (consistent with a single-CME projection), while CME 2 shows a distorted region due to its wider spread of ensemble member positions and orientations
- The results object contains entries for all nRuns ensemble members defined in the .ens file
- Seven averaged-property panels are rendered per CME (impact percentage, duration, Bz, v_r, Kp, density, log T)

## Limitations

- Contour plots and ensemble correlation plots are computationally significant and slow figure generation; README suggests commenting them out for quick testing
- Interpretation depends on ensemble spread: CME 2's distorted nonzero-impact contours arise from wider member position/orientation ranges, so a tight ensemble yields near-single-CME rounded-rectangle shapes
- Results were established for only two case CMEs (22 April and 09 May 2021) with relatively similar coronal origins; more cases are needed to generalize
- Input parameters are not optimized — the paper treats these as real-time-analogous, non-fine-tuned forecasts, so best attainable results are unknown

## Evidence

- [other] For each ensemble member, project the CME footprint at impact (flux rope plus optional sheath) onto a latitude–longitude grid using the member's final position, orientation, size, and shape from ANTEATR/ForeCAT.: [evidence span withheld — anchor status: not_found]
- [readme] This takes the standardized output files, processes them into a results object, then makes an assortment of figures depending on whether a single simulation or a full ensemble was run.: "This takes the standardized output files, processes them into a results object, then makes an assortment of figures depending on whether a single simulation or a full ensemble was run."
- [readme] Others require more significant computation, particularly for the contour plots and ensemble correltation plots, and should potentially be commented out and not run if you want to make quick figures while testing things.: "Others require more significant computation, particularly for the contour plots and ensemble correltation plots, and should potentially be commented out and not run if you want to make quick figures"
- [results] For CME 2, the wider range of ensemble member positions and orientations causes the area of nonzero impact contours to distort from the rounded-rectangle shape, unlike CME 1's nearly unchanged projections.: [evidence span withheld — anchor status: not_found]
- [readme] nRuns - The number of simulation runs. If it is set greater than 1 then the program looks for a second file with the same name as the input file but extension ".ens": "nRuns - The number of simulation runs. If it is set greater than 1 then the program looks for a second file with the same name as the input file but extension ".ens" that sets the parameters varied"
