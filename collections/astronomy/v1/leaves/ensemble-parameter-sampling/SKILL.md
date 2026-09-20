---
name: ensemble-parameter-sampling
description: Use when uncertainty in CME initial parameters (latitude, longitude,
  tilt, coronal velocity, angular width, etc.) must be propagated through ForeCAT/OSPREI
  to produce probabilistic distributions of deflection, rotation, arrival time, or
  in situ properties.
when_to_use_negative:
- When a single deterministic simulation suffices — nRuns defaults to 1 and the .ens
  file is only read when nRuns > 1
- When the goal is optimizing or fine-tuning inputs to achieve a best fit — the article
  explicitly uses real-time-analogous information rather than tuned parameters, and
  notes parameter space exploration is future work
- For slow, streamer-blowout CMEs without changing settings — the 10 Rs FCrmax default
  omits motion occurring at larger distances for such CMEs
edam_operation: http://edamontology.org/operation_0244
tools:
- name: OSPREI
  role: 'Main ensemble driver: reads the text input and .ens file, runs nRuns simulations,
    and generates standardized ensemble outputs processed by processOSPREI.py'
  repo: https://github.com/ckay314/OSPREI
- name: ForeCAT
  role: Component run per ensemble member (via ForeCAT.py) to integrate coronal deflection
    and rotation against the PFSS background out to FCrmax
  repo: https://github.com/ckay314/OSPREI
- name: processOSPREI.py
  role: Loads standardized ensemble output files into a results object and auto-generates
    ensemble profiles, histograms, and correlation plots
  repo: https://github.com/ckay314/OSPREI
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1029/2021sw002914
    title: 'OSPREI: A Coupled Approach to Modeling CME‐Driven Space Weather With Automatically
      Generated, User‐Friendly Outputs'
  source_content_hash: sha256:2ace459ec2c528de992cbf95abca91cab6d648518f9fd9ad9c7c95a3a269d3f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ckay314/OSPREI
  tools:
  - ForeCAT
  - OSPREI
  - processOSPREI.py
  edam_operation: http://edamontology.org/operation_0244
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# ensemble-parameter-sampling

## Summary

Runs OSPREI ensembles by varying input CME parameters across multiple simulation runs (nRuns > 1) to probe input uncertainty and yield probabilities for all outputs, demonstrated with 100-member ForeCAT ensembles for the 09 May and 22 April 2021 CMEs.

## When to use

Use when uncertainty in CME initial parameters (latitude, longitude, tilt, coronal velocity, angular width, etc.) must be propagated through ForeCAT/OSPREI to produce probabilistic distributions of deflection, rotation, arrival time, or in situ properties.

## When NOT to use

- When a single deterministic simulation suffices — nRuns defaults to 1 and the .ens file is only read when nRuns > 1
- When the goal is optimizing or fine-tuning inputs to achieve a best fit — the article explicitly uses real-time-analogous information rather than tuned parameters, and notes parameter space exploration is future work
- For slow, streamer-blowout CMEs without changing settings — the 10 Rs FCrmax default omits motion occurring at larger distances for such CMEs

## Inputs

- OSPREI text input file with seed (Table 1) values and nRuns > 1
- Companion .ens file (same base name) listing varied parameters and their ranges
- PFSS background pickles computed from an HMI synchronic magnetogram (harmonics.py + PFSS.py)

## Outputs

- Per-run final ForeCAT values exported to CSV (one row per ensemble member)
- Ensemble profile plots of latitude, longitude, and tilt versus distance
- Final-value histograms and ensemble correlation/contour plots from processOSPREI.py
- Probabilities/distributions for outputs such as final latitude, longitude, tilt, and deflection magnitude

## How to apply

Write an OSPREI text input file with the required NUMBER parameters (date, time, suffix, CMElat, CMElon in Carrington coordinates, CMEtilt, CMEvr, CMEAW) and set nRuns greater than 1; OSPREI then requires a companion .ens file with the same base name that defines the varied parameters and their ranges. For the 2021 proof-of-concept cases, 100-member (nRuns=100) ForeCAT ensembles varied 17 parameters, using real-time-analogous Table 1 seed values rather than fine-tuned inputs. Ensure the PFSS background pickles (0.5°x0.5°x0.1 Rs grid, 1–2.5 Rs, two ~630 MB files) are built first since ForeCAT interpolates the magnetic field from them for every member. Run the model (ForeCAT.py for FC-only, integrating to FCrmax, default 10 Rs), then run processOSPREI.py to generate ensemble profile plots of latitude, longitude, and tilt versus distance plus final-value histograms, and export per-run final values to CSV.

## Related tools

- **OSPREI** (Main ensemble driver: reads the text input and .ens file, runs nRuns simulations, and generates standardized ensemble outputs processed by processOSPREI.py) — https://github.com/ckay314/OSPREI
- **ForeCAT** (Component run per ensemble member (via ForeCAT.py) to integrate coronal deflection and rotation against the PFSS background out to FCrmax) — https://github.com/ckay314/OSPREI
- **processOSPREI.py** (Loads standardized ensemble output files into a results object and auto-generates ensemble profiles, histograms, and correlation plots) — https://github.com/ckay314/OSPREI

## Examples

```
python OSPREI.py OSPtestcase.txt && python processOSPREI.py OSPtestcase.txt
```

## Evaluation signals

- The .ens file shares the base name of the input text file; otherwise OSPREI cannot find the ensemble specification
- Per-run CSV contains exactly nRuns (e.g., 100) rows of final ForeCAT latitude, longitude, and tilt values
- For CME 1 (09 May 2021), the ensemble mean final latitude, longitude, and tilt fall within 0.5° of the seed values (negligible deflection/rotation)
- For CME 2 (22 April 2021), the mean final deflection is roughly 9° northward and 12° eastward with ~4° counterclockwise rotation
- processOSPREI.py produces the expected ensemble figures: profiles versus distance, final-value histograms, and correlation plots

## Limitations

- Only two CME cases (22 April and 09 May 2021) were tested; more cases are needed to establish the model
- Parameter space was not optimized — best attainable results with tuned inputs are unknown
- Ensemble realism depends on the PFSS background: the synchronic magnetogram does not capture evolution rotating around near the eastern limb
- The ForeCAT Active Region PIL Extraction Routine (ForeCATARPILER/fitPIL) used to seed positions is largely untested beta software

## Evidence

- [intro] OSPREI includes ensemble modeling by varying each input parameter to probe any uncertainty in their values, yielding probabilities for all outputs.: "OSPREI includes ensemble modeling by varying each input parameter to probe any uncertainty in their values, yielding probabilities for all outputs."
- [readme] nRuns controls ensemble mode and .ens file discovery: "If it is set greater than 1 then the program looks for a second file with the same name as the input file but extension ".ens" that sets the parameters varied in the ensemble and their range."
- [readme] ForeCAT interpolates from pickles rather than recalculating: "OSPREI (or at least the ForeCAT portion) will interpolate the background magnetic field from these pickles rather than calculating the vector on the fly during the simulation."
- [other] CME 1 ensemble verification threshold: [evidence span withheld — anchor status: not_found]
