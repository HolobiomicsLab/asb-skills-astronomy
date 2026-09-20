---
name: space-weather-in-situ-property-averaging
description: Use when an OSPREI ensemble run (nRuns > 1 with a .ens file) must be
  summarized as spatial maps showing where the CME impacts the satellite, and how
  averaged in situ properties vary across the lat–lon footprint.
when_to_use_negative:
- Single-run (non-ensemble) simulations — processOSPREI makes a different assortment
  of figures depending on whether a single simulation or a full ensemble was run,
  so per-member spatial averaging does not apply
- Quick exploratory figure generation while testing — the README explicitly suggests
  commenting out the contour plots and ensemble correlation plots because they require
  the most significant computation
- You only need the synthetic spacecraft in situ profile itself (FIDO time series),
  not the spatial distribution of impact probability around the satellite
edam_operation: http://edamontology.org/operation_0337
tools:
- name: OSPREI (processOSPREI.py)
  role: Processes standardized output files into a results object and renders the
    averaged-property contour panels; new figures can be built from the results object
  repo: https://github.com/ckay314/OSPREI
- name: ForeCAT
  role: Supplies each ensemble member's final latitude, longitude, and tilt at the
    end of the corona, which fix the projected footprint's position and orientation
    on the lat–lon grid
  repo: https://github.com/ckay314/OSPREI
- name: ANTEATR
  role: Supplies final CME speed, size, shape, and internal thermal/magnetic properties
    at the final satellite distance, which set the projected footprint extent
  repo: https://github.com/ckay314/OSPREI
- name: FIDO
  role: Provides the in situ magnetic field, velocity, and Kp values that are averaged
    per lat–lon bin across ensemble members
  repo: https://github.com/ckay314/OSPREI
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1029/2021sw002914
    title: 'OSPREI: A Coupled Approach to Modeling CME‐Driven Space Weather With Automatically
      Generated, User‐Friendly Outputs'
  source_content_hash: sha256:75f2eae6314497124e76a9e471a6bf662b6e3deda236f53ad00b87ad48cd6f34
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ckay314/OSPREI
  tools:
  - ANTEATR
  - FIDO
  - ForeCAT
  - OSPREI (processOSPREI.py)
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# space-weather-in-situ-property-averaging

## Summary

Generates Figure-8-style latitude–longitude contour maps around a satellite by averaging ensemble-member CME impact probability and in situ properties (duration, Bz, v_r, Kp, density, log T) across all nRuns OSPREI ensemble members, using the standardized figure routines in processOSPREI.py.

## When to use

Use when an OSPREI ensemble run (nRuns > 1 with a .ens file) must be summarized as spatial maps showing where the CME impacts the satellite, and how averaged in situ properties vary across the lat–lon footprint.

## When NOT to use

- Single-run (non-ensemble) simulations — processOSPREI makes a different assortment of figures depending on whether a single simulation or a full ensemble was run, so per-member spatial averaging does not apply
- Quick exploratory figure generation while testing — the README explicitly suggests commenting out the contour plots and ensemble correlation plots because they require the most significant computation
- You only need the synthetic spacecraft in situ profile itself (FIDO time series), not the spatial distribution of impact probability around the satellite

## Inputs

- OSPREI standardized ensemble output files (ForeCAT final position/tilt, ANTEATR final size/shape/speed, FIDO in situ properties), grouped by date and suffix
- .ens ensemble file listing the varied parameters, their ranges, and nRuns
- OSPREI input text file (date, time, suffix) passed to processOSPREI.py

## Outputs

- Seven averaged-property latitude–longitude contour panels per CME (impact percentage, mean duration, Bz, v_r, Kp, density, log T)
- Figure files saved under the standardized date- and suffix-based naming convention

## How to apply

Run processOSPREI.py on the standardized OSPREI output files so they are processed into a results object holding every ensemble member's CME geometry and properties at impact. For each of the nRuns members defined by the .ens ensemble file, project the CME footprint at impact (flux rope plus optional sheath) onto a latitude–longitude grid using the member's final position, orientation, size, and shape produced by ForeCAT/ANTEATR. Accumulate per-bin ensemble statistics — impact percentage, mean duration, Bz, v_r, Kp, density, and log T — across all members, then render the seven averaged-property contour panels per CME with the matplotlib-based standardized figure routines. Note that the README warns the contour plots are the most computation-heavy outputs and can be commented out when only quick test figures are needed. Save the panel figure files using OSPREI's date- and suffix-based naming convention so multiple CMEs or configurations are not overwritten.

## Related tools

- **OSPREI (processOSPREI.py)** (Processes standardized output files into a results object and renders the averaged-property contour panels; new figures can be built from the results object) — https://github.com/ckay314/OSPREI
- **ForeCAT** (Supplies each ensemble member's final latitude, longitude, and tilt at the end of the corona, which fix the projected footprint's position and orientation on the lat–lon grid) — https://github.com/ckay314/OSPREI
- **ANTEATR** (Supplies final CME speed, size, shape, and internal thermal/magnetic properties at the final satellite distance, which set the projected footprint extent) — https://github.com/ckay314/OSPREI
- **FIDO** (Provides the in situ magnetic field, velocity, and Kp values that are averaged per lat–lon bin across ensemble members) — https://github.com/ckay314/OSPREI

## Examples

```
python processOSPREI.py OSPtestcase.txt
```

## Evaluation signals

- All impact-percentage values fall within [0, 100]% in the accumulated bins
- For a case like CME 1 (narrow ensemble spread), the nonzero-impact contour region is a rounded rectangle matching a single CME projection; wider position/orientation spreads (CME 2) produce a distorted region
- Exactly seven averaged-property contour panels are produced per CME (impact %, duration, Bz, v_r, Kp, density, log T)
- Figure files exist under the standardized date- and suffix-based naming convention without overwriting other CMEs or configurations
- Expert review confirms the maps reproduce the expected Figure 8 pattern (rounded-rectangle vs. distorted nonzero-impact region)

## Limitations

- The contour plots are the most computationally significant figures; the README advises commenting them out when quick figures are needed during testing
- Only two CME cases were considered in the study; more cases are needed to establish whether behaviors like the distorted impact region generalize
- FIDO-SIT can only assume a Parker Spiral orientation for the upstream direction and cannot predict actual in situ magnetic field components, limiting the fidelity of field-component maps
- Initial CME position routines like ForeCATARPILER (fitPIL) are beta and largely untested, so ensemble-member initial geometry may need visual verification

## Evidence

- [readme] This takes the standardized output files, processes them into a results object, then makes an assortment of figures depending on whether a single simulation or a full ensemble was run.: "This takes the standardized output files, processes them into a results object, then makes an assortment of figures depending on whether a single simulation or a full ensemble was run."
- [readme] Others require more significant computation, particularly for the contour plots and ensemble correltation plots, and should potentially be commented out and not run: "Others require more significant computation, particularly for the contour plots and ensemble correltation plots, and should potentially be commented out and not run"
- [intro] OSPREI automatically generates standardized visualizations providing accessible information for space weather forecasting: [evidence span withheld — anchor status: not_found]
- [readme] A suffix name to add as a subscript to the save files; OSPREI groups files by date so multiple CMEs or configurations do not overwrite each other.: "OSPREI automatically generates file names based on the date and groups them in folders. This allows for different configurations for the same case or multiple CMEs on the same day with different tags"
