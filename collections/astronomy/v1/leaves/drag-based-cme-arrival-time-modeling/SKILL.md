---
name: drag-based-cme-arrival-time-modeling
description: Use when a coronagraph- or ForeCAT-derived far-corona CME state (lat,
  lon, tilt, coronal speed) must be propagated to 1 AU to predict transit/arrival
  time, duration, density, and central B at a spacecraft.
when_to_use_negative:
- When the goal is coronal deflection and rotation — use ForeCAT, since little deflection
  occurs beyond the default FCrmax=10 Rs where ForeCAT ends and ANTEATR begins
- When synthetic in situ magnetic field/velocity profiles or Kp index are needed —
  run FIDO in addition to (not instead of) ANTEATR
- When forecasting relies only on hydrodynamic properties with no internal magnetic
  structure — ANTEATR-PARADE is specifically used to add expansion, deformation, and
  internal property evolution
edam_operation: http://edamontology.org/operation_2426
tools:
- name: OSPREI
  role: Coupled suite; OSPREI.py is invoked with an input text file to run ANTEATR
    from the far-corona state
  repo: https://github.com/ckay314/OSPREI
- name: ForeCAT
  role: Provides the terminal coronal CME state (final latitude, longitude, tilt,
    speed) that initializes interplanetary propagation
  repo: https://github.com/ckay314/OSPREI
- name: ANTEATR-PARADE
  role: Drag-based interplanetary propagation with expansion and deformation, fully
    contained in PARADE.py; yields arrival time and internal properties
  repo: https://github.com/ckay314/OSPREI
- name: processOSPREI.py
  role: Processes standardized output files into a results object and generates histograms
    and profile figures
  repo: https://github.com/ckay314/OSPREI
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1029/2021sw002914
    title: 'OSPREI: A Coupled Approach to Modeling CME‐Driven Space Weather With Automatically
      Generated, User‐Friendly Outputs'
  source_content_hash: sha256:275f2f162f7afd7e26d9446d2f7cfd6e2ac9c3cf3c014f86dffc11f9d0fe5a2b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ckay314/OSPREI
  tools:
  - ANTEATR
  - ANTEATR-PARADE
  - ForeCAT
  - OSPREI
  - processOSPREI.py
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# Drag-based CME arrival time modeling

## Summary

Reproduce ANTEATR-PARADE interplanetary CME propagation in OSPREI, which applies a drag force toward the background solar wind plus flux-rope expansion/deformation to predict arrival time and internal magnetic/thermal properties at a satellite distance.

## When to use

Use when a coronagraph- or ForeCAT-derived far-corona CME state (lat, lon, tilt, coronal speed) must be propagated to 1 AU to predict transit/arrival time, duration, density, and central B at a spacecraft.

## When NOT to use

- When the goal is coronal deflection and rotation — use ForeCAT, since little deflection occurs beyond the default FCrmax=10 Rs where ForeCAT ends and ANTEATR begins
- When synthetic in situ magnetic field/velocity profiles or Kp index are needed — run FIDO in addition to (not instead of) ANTEATR
- When forecasting relies only on hydrodynamic properties with no internal magnetic structure — ANTEATR-PARADE is specifically used to add expansion, deformation, and internal property evolution

## Inputs

- OSPREI input text file with date, time, suffix, and models=ANT
- ForeCAT terminal CME state (final latitude, longitude, tilt, coronal speed CMEvr) loaded into PARADE.py
- Background solar wind parameters (SWCd=1, SWv=400 km/s, SWn=5 cm^-3, SWB=6.9 nT)
- Flux rope settings (FRpol, FRB/FRT scaled from CME speed, Gamma=1.33, IVDf=0.5)

## Outputs

- Mean ensemble arrival (transit) times at the satellite distance (e.g., 12 May 2021 18:28 UT for CME 1; 25 April 2021 01:47 UT for CME 2)
- Profiles vs distance of propagation and expansion speeds, angular widths, aspect ratios
- Internal thermal and magnetic properties at arrival (mean durations 36 and 27 hr; densities 11.1 and 5.8 cm^-3; central B 18 vs 8 nT)
- Standardized OSPREI output files processable by processOSPREI.py into histograms and profile figures

## How to apply

Prepare an OSPREI input text file with date (YYYYMMDD), time (HH:MM), a required suffix, and models=ANT so ANTEATR alone runs from the far-corona state, keeping satellite defaults SatLat=0°, SatLon=0°, SatR=213 Rs. Load the ForeCAT terminal CME state (final latitude, longitude, tilt, coronal speed) into PARADE.py as the initial condition for interplanetary propagation. Run ANTEATR-PARADE with drag toward the background solar wind (SWCd=1, SWv=400 km/s, SWn=5 cm^-3, SWB=6.9 nT — OMNI-average defaults at 1 AU) and flux rope settings FRpol, FRB/FRT scaled from the CME speed, Gamma=1.33, and IVDf=0.5. Record the evolution with distance of propagation/expansion speeds, angular widths, aspect ratios, and internal magnetic field and temperature through to arrival; then process standardized output files with processOSPREI.py and inspect histograms and profile figures.

## Related tools

- **OSPREI** (Coupled suite; OSPREI.py is invoked with an input text file to run ANTEATR from the far-corona state) — https://github.com/ckay314/OSPREI
- **ForeCAT** (Provides the terminal coronal CME state (final latitude, longitude, tilt, speed) that initializes interplanetary propagation) — https://github.com/ckay314/OSPREI
- **ANTEATR-PARADE** (Drag-based interplanetary propagation with expansion and deformation, fully contained in PARADE.py; yields arrival time and internal properties) — https://github.com/ckay314/OSPREI
- **processOSPREI.py** (Processes standardized output files into a results object and generates histograms and profile figures) — https://github.com/ckay314/OSPREI

## Examples

```
python OSPREI.py OSPtestcase.txt && python processOSPREI.py OSPtestcase.txt
```

## Evaluation signals

- Emitted CSV contains mean transit times ~18:28 UT on 12 May 2021 (CME 1) and ~01:47 UT on 25 April 2021 (CME 2), each value_in_range of reported values
- Mean durations fall at ~36 hr (CME 1) and ~27 hr (CME 2)
- Mean densities fall at ~11.1 and 5.8 cm^-3 and central B at ~18 vs 8 nT, within reported ranges
- Predicted arrival compares plausibly to observed magnetic obstacle arrivals (22:00 UT on 12 May 2021 for CME 1)
- processOSPREI.py successfully builds a results object and generates histograms of profile quantities

## Limitations

- Simple distance/speed arrival estimates lead to large errors because they neglect drag — hence drag must be included, but drag depends on an assumed background solar wind
- The background solar wind is the most under-developed component of OSPREI; effects of different backgrounds are untested
- Only two CME cases (22 April and 09 May 2021) were considered; parameter space was not optimized and inputs were not fine-tuned

## Evidence

- [readme] ANTEATR takes the ForeCAT CME and propagates it to the final satellite distance. Like most arrival time models, ANTEATR includes a drag force that causes the CME speed to approach that of the background solar wind.: "ANTEATR takes the ForeCAT CME and propagates it to the final satellite distance. Like most arrival time models, ANTEATR includes a drag force that causes the CME speed to approach that of the"
- [methods] ANTEATR-PARADE flux rope and drag settings: [evidence span withheld — anchor status: not_found]
- [methods] OSPREI defaults correspond to OMNI averages: "All solar wind properties have defaults corresponding to the average OMNI values at 1 AU"
- [intro] Drag omission error rationale: "an arrival time estimated using simply the distance and speed, but this method leads to large errors as it neglects the change in CME speed due to drag"
- [discussion] Background solar wind limitation: [evidence span withheld — anchor status: not_found]
