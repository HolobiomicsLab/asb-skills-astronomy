---
name: cme-footprint-geometry-interpretation
description: Use when a latitude–longitude contour map of ensemble CME impact percentage
  shows a distorted (non–rounded-rectangle) nonzero-impact region and you need to
  attribute the distortion to ensemble spread in CME position, orientation, size,
  or shape at impact.
when_to_use_negative:
- When working with single-run (deterministic) OSPREI output rather than a full ensemble
  — the rounded-rectangle vs. distorted-footprint diagnosis depends on the ensemble
  spread and does not apply to one simulation
- When you only need arrival time or radial profiles/histograms from processOSPREI.py
  and want to skip the computationally significant contour-plot computation
- When the question concerns real-time forecast accuracy benchmarks (e.g., WSA-ENLIL
  + Cone arrival time errors) rather than ensemble footprint geometry
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: OSPREI
  role: Suite wrapper whose processOSPREI.py builds the results object and renders
    the standardized ensemble contour panels (Figure 8) with matplotlib
  repo: https://github.com/ckay314/OSPREI
- name: ForeCAT
  role: Provides each ensemble member's final latitude, longitude, and tilt in the
    outer corona, which determine the projected footprint position and orientation
  repo: https://github.com/ckay314/OSPREI
- name: ANTEATR
  role: Provides the CME's final speed, size, shape, and arrival time at the satellite
    distance, defining the footprint extent at impact
  repo: https://github.com/ckay314/OSPREI
- name: FIDO
  role: Supplies the in situ magnetic field, velocity, and Kp values averaged into
    the contour panels over the impacted latitude–longitude bins
  repo: https://github.com/ckay314/OSPREI
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1029/2021sw002914
    title: 'OSPREI: A Coupled Approach to Modeling CME‐Driven Space Weather With Automatically
      Generated, User‐Friendly Outputs'
  source_content_hash: sha256:c98b1fab093454e276d445e32f0be4e3442161155414e05809e36fc788e56c5b
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
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# cme-footprint-geometry-interpretation

## Summary

Interprets OSPREI ensemble latitude–longitude contour maps of CME impact probability and in situ properties (Figure 8 of Kay et al. 2022), explaining that the nonzero-impact footprint deviates from a rounded rectangle when ensemble members spread widely in final position, orientation, size, and shape.

## When to use

Use when a latitude–longitude contour map of ensemble CME impact percentage shows a distorted (non–rounded-rectangle) nonzero-impact region and you need to attribute the distortion to ensemble spread in CME position, orientation, size, or shape at impact.

## When NOT to use

- When working with single-run (deterministic) OSPREI output rather than a full ensemble — the rounded-rectangle vs. distorted-footprint diagnosis depends on the ensemble spread and does not apply to one simulation
- When you only need arrival time or radial profiles/histograms from processOSPREI.py and want to skip the computationally significant contour-plot computation
- When the question concerns real-time forecast accuracy benchmarks (e.g., WSA-ENLIL + Cone arrival time errors) rather than ensemble footprint geometry

## Inputs

- OSPREI standardized ensemble output files for the case CMEs (from running OSPREI.py with a .txt input file and a matching .ens ensemble file)
- .ens ensemble file specifying the varied input parameters and their ranges
- nRuns (number of ensemble members) from the OSPREI input file

## Outputs

- Seven-panel latitude–longitude contour figure per CME (impact percentage, mean duration, Bz, v_r, Kp, density, log T)
- Figure files saved under the OSPREI date- and suffix-based standardized naming convention
- Interpretation of why the nonzero-impact region deviates from a rounded rectangle (ensemble spread in position/orientation/size/shape)

## How to apply

Load the standardized OSPREI ensemble outputs with processOSPREI.py, which builds a results object holding every ensemble member's CME geometry (final position, orientation, size, shape from ForeCAT/ANTEATR) and internal properties at impact. Project each member's footprint at impact — flux rope plus optional sheath — onto a latitude–longitude grid, then accumulate per-bin ensemble statistics (impact percentage, mean duration, Bz, v_r, Kp, density, log T) across the nRuns members defined by the .ens ensemble file. Render the seven averaged-property contour panels per CME using the OSPREI standardized figure routines, noting that contour plots are among the most computationally significant figures and can be skipped for quick tests. Rationale: a single CME projects as a rounded rectangle in latitude–longitude, but when the ensemble spans a wide range of member positions and orientations, the union of footprints distorts the nonzero-impact contour — as seen for CME 2, whereas CME 1's projections remain nearly unchanged. Compare the distortion between CMEs by checking whether the impact-region boundary tracks the spread of ForeCAT final latitudes/longitudes/tilts rather than any single footprint.

## Related tools

- **OSPREI** (Suite wrapper whose processOSPREI.py builds the results object and renders the standardized ensemble contour panels (Figure 8) with matplotlib) — https://github.com/ckay314/OSPREI
- **ForeCAT** (Provides each ensemble member's final latitude, longitude, and tilt in the outer corona, which determine the projected footprint position and orientation) — https://github.com/ckay314/OSPREI
- **ANTEATR** (Provides the CME's final speed, size, shape, and arrival time at the satellite distance, defining the footprint extent at impact) — https://github.com/ckay314/OSPREI
- **FIDO** (Supplies the in situ magnetic field, velocity, and Kp values averaged into the contour panels over the impacted latitude–longitude bins) — https://github.com/ckay314/OSPREI

## Examples

```
python processOSPREI.py OSPtestcase.txt
```

## Evaluation signals

- Each rendered map figure file exists under the OSPREI date- and suffix-based standardized naming convention for both case CMEs
- All impact-percentage values in the contour maps fall within [0, 100]%
- CME 1's nonzero-impact region appears as a rounded rectangle (ensemble footprints nearly unchanged), consistent with its narrow ensemble spread
- CME 2's nonzero-impact region shows a distorted shape, and the distortion aligns with the wider range of ForeCAT final positions and orientations among ensemble members
- All seven averaged-property panels (impact %, duration, Bz, v_r, Kp, density, log T) render per CME across the nRuns members of the .ens ensemble

## Limitations

- The interpretation was demonstrated on only two case CMEs (22 April and 09 May 2021); more cases are needed to establish generality
- The parameter space was not optimized — inputs were deliberately not fine-tuned (forecasting proof-of-concept), so the footprint geometry reflects those untuned ensemble ranges
- If the ensemble spread in position/orientation is small (as for CME 1), contour distortion is minimal and the diagnostic has little discriminating power

## Evidence

- [abstract] Ensemble modeling by varying each input parameter to probe uncertainty, yielding probabilities for all outputs: "OSPREI includes ensemble modeling by varying each input parameter to probe any uncertainty in their values, yielding probabilities for all outputs."
- [intro] Standardized visualizations automatically generated for forecasting: "Standardized visualizations are automatically generated, providing easily accessible, essential information for space weather forecasting."
- [readme] processOSPREI.py builds results object and makes figures for ensemble runs: "This takes the standardized output files, processes them into a results object, then makes an assortment of figures depending on whether a single simulation or a full ensemble was run."
- [readme] Contour plots are the most computation-intensive figures: "Others require more significant computation, particularly for the contour plots and ensemble correltation plots, and should potentially be commented out and not run if you want to make quick figures"
- [readme] ForeCAT output is final latitude, longitude, tilt of the CME: "The output of the model is the final latitude, longitude, and tilt of the CME in the outer corona (as well as the profiles of these parameters versus distance)."
