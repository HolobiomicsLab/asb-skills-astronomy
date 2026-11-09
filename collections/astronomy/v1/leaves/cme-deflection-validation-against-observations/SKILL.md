---
name: cme-deflection-validation-against-observations
description: Use when validating that ForeCAT reproduces known coronal deflection/rotation
  behavior for specific CMEs — e.g., a near-radial CME should stay within ~0.5° of
  its seed latitude/longitude/tilt, while a deflecting CME shows multi-degree latitudinal
  and longitudinal displacement.
when_to_use_negative:
- You need interplanetary propagation, arrival time, or in situ profiles — set models
  to ALL (or IP) rather than FC-only; ForeCAT ends at FCrmax (10 Rs default) and cannot
  address 1 AU behavior
- The case is a slow, streamer blowout CME with significant motion beyond 10 Rs —
  the FCrmax=10 Rs default is inappropriate per the README and must be increased
- No PFSS background exists for the eruption day and you cannot afford the ~30–60
  min PFSS.py run — ForeCAT cannot run without the pickled magnetic field
edam_operation: http://edamontology.org/operation_0244
tools:
- name: OSPREI
  role: Main suite invoked with a text input file and .ens ensemble file; provides
    processOSPREI.py for figures and results object
  repo: https://github.com/ckay314/OSPREI
- name: ForeCAT
  role: Coronal model (ForeCAT.py, CME_class.py, ForceFields.py) that computes deflection
    and rotation against the PFSS background; run via models=FC
  repo: https://github.com/ckay314/OSPREI
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1029/2021sw002914
    title: 'OSPREI: A Coupled Approach to Modeling CME‐Driven Space Weather With Automatically
      Generated, User‐Friendly Outputs'
  source_content_hash: sha256:dd6f7ebf07783d7f8580d85f7951a24291e511da13067bda74cc017fe0982018
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ckay314/OSPREI
  tools:
  - ForeCAT
  - OSPREI
  edam_operation: http://edamontology.org/operation_0244
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cme-deflection-validation-against-observations

## Summary

Runs ForeCAT (the coronal component of OSPREI) as a 100-member ensemble against a PFSS magnetic background to quantify coronal CME deflection and rotation, then validates the mean final latitude, longitude, and tilt against expected values (e.g., <0.5° from seed for undeflecting CMEs, ~9°N/12°E deflection for deflecting CMEs).

## When to use

Use when validating that ForeCAT reproduces known coronal deflection/rotation behavior for specific CMEs — e.g., a near-radial CME should stay within ~0.5° of its seed latitude/longitude/tilt, while a deflecting CME shows multi-degree latitudinal and longitudinal displacement.

## When NOT to use

- You need interplanetary propagation, arrival time, or in situ profiles — set models to ALL (or IP) rather than FC-only; ForeCAT ends at FCrmax (10 Rs default) and cannot address 1 AU behavior
- The case is a slow, streamer blowout CME with significant motion beyond 10 Rs — the FCrmax=10 Rs default is inappropriate per the README and must be increased
- No PFSS background exists for the eruption day and you cannot afford the ~30–60 min PFSS.py run — ForeCAT cannot run without the pickled magnetic field

## Inputs

- HMI synchronic magnetogram fits file for the eruption day (JSOC catalog, hmi.Mrdailysynframe_720s)
- OSPREI text input file with REQUIRED parameters: date (YYYYMMDD), time (HH:MM), suffix, CMElat, CMElon (Carrington), CMEtilt, CMEvr, CMEAW
- Companion .ens file with same base name setting nRuns=100 and defining the 17 varied ensemble parameters and ranges
- PFSS pickles (two ~630 MB files from PFSS.py) containing the 1–2.5 Rs magnetic field vector

## Outputs

- Per-run final ForeCAT latitude, longitude, and tilt values exported to CSV
- Ensemble profile plots of latitude, longitude, and tilt versus distance and final-value histograms from processOSPREI.py
- Standardized OSPREI output files grouped by date and suffix

## How to apply

First build the PFSS background once per case: acquire an HMI synchronic magnetogram from JSOC, run sync2synop.py to shift it to the Carrington frame, then harmonics.py for harmonic coefficients and PFSS.py to compute the magnetic field on a 0.5°x0.5°x0.1 Rs grid between 1 and 2.5 Rs (two ~630 MB pickles). Next, write an OSPREI text input file per CME with date, time, suffix, CMElat, CMElon (Carrington coordinates), CMEtilt, CMEvr, CMEAW from Table 1, and set models=FC so only ForeCAT runs; create the companion .ens file with nRuns=100 and the 17 varied parameter ranges. Run ForeCAT.py, which interpolates the background field from the pickles and integrates each member's deflection and rotation out to FCrmax (default 10 Rs). Finally run processOSPREI.py to build the results object with profile plots of latitude, longitude, and tilt versus distance plus final-value histograms, and export per-run final ForeCAT values to CSV for verification.

## Related tools

- **OSPREI** (Main suite invoked with a text input file and .ens ensemble file; provides processOSPREI.py for figures and results object) — https://github.com/ckay314/OSPREI
- **ForeCAT** (Coronal model (ForeCAT.py, CME_class.py, ForceFields.py) that computes deflection and rotation against the PFSS background; run via models=FC) — https://github.com/ckay314/OSPREI

## Examples

```
python OSPREI.py OSPtestcase.txt && python processOSPREI.py OSPtestcase.txt
```

## Evaluation signals

- For CME 1 (09 May 2021), the ensemble mean final latitude, longitude, AND tilt all fall within 0.5° of the ensemble seed values
- For CME 2 (22 April 2021), the ensemble mean deflection is roughly 9° northward and 12° eastward, with negligible (~4° counterclockwise) rotation
- Both ~630 MB PFSS pickles exist before running ForeCAT.py, and the run completes to FCrmax (10 Rs) without interpolation errors
- The .ens companion file is found (same base name as the input .txt) and nRuns=100 runs appear in the standardized output files and CSV export
- processOSPREI.py generates ensemble profile plots (latitude/longitude/tilt vs distance) and final-value histograms for each case

## Limitations

- ForeCAT default FCrmax of 10 Rs is unsuitable for slow, streamer blowout CMEs, which continue deflecting beyond that distance
- PFSS 'ringing' effects appear at heights below ~1.05 Rs when visualizing the PIL in selectregion.py; higher heights may be needed for quiet sun/streamer belt/stealth CMEs
- The HMI synchronic magnetogram does not capture solar evolution rotating around near the eastern limb, so eastern active regions are imperfectly represented
- The ForeCATARPILER/fitPIL routine for automatic PIL extraction is 'largely untested and should be considered a beta program'

## Evidence

- [readme] OSPREI runs ForeCAT for coronal CME deflection and rotation, ANTEATR (PARADE) for interplanetary CME propagation, expansion, and deformation: "OSPREI runs ForeCAT for coronal CME deflection and rotation, ANTEATR (PARADE) for interplanetary CME propagation, expansion, and deformation"
- [readme] PFSS grid resolution and extent: "This script will take some time to run as it calculated the magnetic field vector on a 0.5°x0.5°x0.1 Rs grid between 1 Rs and 2.5 Rs"
- [readme] FCrmax default and rationale: "The radial distance at which ForeCAT ends and ANTEATR begins. We default to 10 Rs for "normal" CMEs as little deflection or rotation tends to happen beyond this distances."
- [readme] Ensemble mechanism: "nRuns - The number of simulation runs. If it is set greater than 1 then the program looks for a second file with the same name as the input file but extension ".ens""
- [intro] ForeCAT role in the Sun-to-Earth chain: "First, Forecasting a CME's Altered Trajectory (ForeCAT) describes the trajectory that a CME takes through the solar corona."
- [discussion] Coupled workflow order: "OSPREI first performs a ForeCAT simulation of the coronal deflection and rotation of each CME."
