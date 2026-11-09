---
name: cme-coronal-deflection-rotation-simulation
description: Use when a study needs the coronal trajectory of one or more CMEs — final
  latitude, longitude, and tilt plus profiles vs. distance — from a magnetogram-derived
  PFSS background, with optional ensemble uncertainty quantification.
when_to_use_negative:
- The goal is interplanetary arrival time, expansion, or in situ profiles — use the
  ANTEATR (ANT) or FIDO components, or run models=ALL, rather than FC alone
- 'The CME of interest is a slow, streamer-blowout event: the default FCrmax of 10
  Rs should not be used and ForeCAT''s coronal window must be extended, so the stock
  configuration is inappropriate'
- No suitable magnetogram/PFSS background exists for the eruption day and building
  one (30–60 min PFSS.py run) is infeasible
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: OSPREI
  role: Parent suite; OSPREI.py invoked with a text input file, models=FC restricts
    execution to ForeCAT, and processOSPREI.py generates the figures
  repo: https://github.com/ckay314/OSPREI
- name: ForeCAT
  role: 'Core model: uses the PFSS background to compute external magnetic forces
    causing CME deflection and net torque-driven rotation out to FCrmax'
  repo: https://github.com/ckay314/OSPREI
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1029/2021sw002914
    title: 'OSPREI: A Coupled Approach to Modeling CME‐Driven Space Weather With Automatically
      Generated, User‐Friendly Outputs'
  source_content_hash: sha256:823222a974f6580e15432daa9221b8a957fe062298da13699d5e748be34fd628
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ckay314/OSPREI
  tools:
  - ForeCAT
  - OSPREI
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cme-coronal-deflection-rotation-simulation

## Summary

Runs ForeCAT, the coronal component of OSPREI, to simulate how a CME deflects in latitude/longitude and rotates in tilt as it propagates through the PFSS-modeled background magnetic field out to FCrmax (default 10 Rs), optionally as an nRuns>1 ensemble.

## When to use

Use when a study needs the coronal trajectory of one or more CMEs — final latitude, longitude, and tilt plus profiles vs. distance — from a magnetogram-derived PFSS background, with optional ensemble uncertainty quantification.

## When NOT to use

- The goal is interplanetary arrival time, expansion, or in situ profiles — use the ANTEATR (ANT) or FIDO components, or run models=ALL, rather than FC alone
- The CME of interest is a slow, streamer-blowout event: the default FCrmax of 10 Rs should not be used and ForeCAT's coronal window must be extended, so the stock configuration is inappropriate
- No suitable magnetogram/PFSS background exists for the eruption day and building one (30–60 min PFSS.py run) is infeasible

## Inputs

- HMI synchronic magnetogram fits file (HMIYYYYMMDDsync.fits) from the JSOC catalog for the eruption day
- OSPREI text input file with REQUIRED parameters: date, time, suffix, CMElat, CMElon (Carrington), CMEtilt, CMEvr, CMEAW, plus models=FC and FCrmax
- Companion .ens file (same base name) setting nRuns and the varied ensemble parameters and ranges
- PFSS pickle files (~630 MB each) containing the magnetic field vector on the 0.5°x0.5°x0.1 Rs grid, 1–2.5 Rs

## Outputs

- Standardized ForeCAT output files with per-run final latitude, longitude, and tilt at FCrmax and profiles of these parameters versus distance
- Figures from processOSPREI.py: ensemble profiles of latitude/longitude/tilt vs. distance and final-value histograms
- Exportable per-run final ForeCAT values (CSV)

## How to apply

First build the magnetic background: download an HMI synchronic magnetogram for the eruption day from JSOC, shift it to the Carrington frame with sync2synop.py, run harmonics.py to compute harmonic coefficients, then PFSS.py to compute the magnetic field vector on a 0.5°x0.5°x0.1 Rs grid between 1 and 2.5 Rs, which saves two ~630 MB pickles. Next, write an OSPREI text input file setting date (YYYYMMDD), time, suffix, CMElat, CMElon (in Carrington coordinates so the CME is placed correctly in the PFSS background), CMEtilt, CMEvr, CMEAW, and models=FC so only ForeCAT runs. For an ensemble, set nRuns (e.g., 100) and create a companion .ens file with the same base name defining the varied parameters and their ranges. Run ForeCAT.py, which interpolates the background field from the pickles and integrates each member's deflection and rotation out to FCrmax (10 Rs by default, since little deflection/rotation occurs beyond this for normal CMEs; use a larger value for slow, streamer-blowout CMEs). Finally, process outputs with processOSPREI.py to generate latitude/longitude/tilt-versus-distance profiles and final-value histograms.

## Related tools

- **OSPREI** (Parent suite; OSPREI.py invoked with a text input file, models=FC restricts execution to ForeCAT, and processOSPREI.py generates the figures) — https://github.com/ckay314/OSPREI
- **ForeCAT** (Core model: uses the PFSS background to compute external magnetic forces causing CME deflection and net torque-driven rotation out to FCrmax) — https://github.com/ckay314/OSPREI

## Examples

```
python OSPREI.py OSP_CME1_FC.txt && python processOSPREI.py OSP_CME1_FC.txt
```

## Evaluation signals

- PFSS.py produced two ~630 MB pickles covering 1–2.5 Rs on the 0.5°x0.5°x0.1 Rs grid; ForeCAT interpolates from these rather than recomputing the field
- All output files present in CSV and figure form for every ensemble run after processOSPREI.py
- In the published case, the CME 1 ensemble mean final latitude, longitude, and tilt fell within 0.5° of the ensemble seed values (negligible deflection/rotation)
- CME 2 ensemble reproduced mean deflection of ~9° northward and ~12° eastward with only ~4° counterclockwise rotation
- Ensemble profiles of latitude, longitude, and tilt vs. distance and final-value histograms generated without errors from the standardized output files

## Limitations

- The default FCrmax of 10 Rs assumes 'normal' CMEs; slow, streamer-blowout CMEs exhibit more motion at larger distances and need a larger FCrmax
- CMElon must be in the Carrington coordinate frame at the simulation start time, or the CME will be misplaced in the PFSS background
- The synchronic magnetogram does not capture solar evolution rotating around near the eastern limb, where active regions may appear
- The ForeCATARPILER/fitPIL routine for initial position is largely untested and should be considered a beta program
- Only two CME cases (22 April and 09 May 2021) were validated; parameter space was not optimized or fine-tuned

## Evidence

- [readme] PFSS grid and pickles: "This script will take some time to run as it calculated the magnetic field vector on a 0.5°x0.5°x0.1 Rs grid between 1 Rs and 2.5 Rs"
- [readme] FCrmax default and exception: "We default to 10 Rs for "normal" CMEs as little deflection or rotation tends to happen beyond this distances. Slow, streamer blowout CMEs exhibit more motion at farther distances"
- [readme] Carrington longitude requirement: "If running ForeCAT, this needs to be in the Carrington coordinates reference frame at the time corresponding to the start of the simulation to place the CME in the PFSS background."
- [readme] ForeCAT computes deflection and rotation from forces: "ForeCAT uses the PFSS background to determine the external magnetic forces on a CME. These forces cause the CME to deflect in latitude and longitude and the net torque determines a rotation."
- [results] Published ensemble result for both CMEs: [evidence span withheld — anchor status: not_found]
- [methods] Magnetogram recommendation: "We recommend using an HMI synchronic magnetogram for the day of the eruption"
- [intro] Ensemble modeling purpose: "OSPREI includes ensemble modeling by varying each input parameter to probe any uncertainty in their values, yielding probabilities for all outputs."
