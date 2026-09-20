---
name: cme-interplanetary-propagation-simulation
description: Use when you have a ForeCAT far-corona terminal CME state (latitude,
  longitude, tilt, coronal speed) and need to propagate it through interplanetary
  space to predict arrival time, speed, size, and internal magnetic/thermal properties
  at a satellite.
when_to_use_negative:
- The CME has not yet been simulated through the corona and you need deflection/rotation
  from the solar surface — run ForeCAT first, since ANTEATR takes the ForeCAT terminal
  state as its initial condition
- You need synthetic in situ profiles at a spacecraft (magnetic field vector, velocity,
  Kp) — that is FIDO's role; ANTEATR only propagates the CME and gives bulk/arrival
  properties
- You only need a quick coronagraph-based arrival time estimate without expansion,
  deformation, or internal properties — simple drag-based or WSA-ENLIL+Cone arrival
  models may suffice
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_3360
tools:
- name: OSPREI
  role: Coupled Sun-to-Earth CME modeling suite; invoked with models=ANT to run ANTEATR
    alone from the far-corona state
  repo: https://github.com/ckay314/OSPREI
- name: ANTEATR-PARADE
  role: Interplanetary propagation model (fully contained in PARADE.py) with drag,
    expansion, and deformation; takes the ForeCAT CME and propagates it to the final
    satellite distance
  repo: https://github.com/ckay314/OSPREI
- name: ForeCAT
  role: Provides the terminal far-corona CME state (final latitude, longitude, tilt,
    coronal speed) that seeds ANTEATR-PARADE
  repo: https://github.com/ckay314/OSPREI
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1029/2021sw002914
    title: 'OSPREI: A Coupled Approach to Modeling CME‐Driven Space Weather With Automatically
      Generated, User‐Friendly Outputs'
  source_content_hash: sha256:0ecddc6e3860790fef76fe991d0fae109fe84c076922c90bc69009aa043f1b9b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ckay314/OSPREI
  tools:
  - ANTEATR
  - ANTEATR-PARADE
  - ForeCAT
  - OSPREI
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# cme-interplanetary-propagation-simulation

## Summary

Simulates interplanetary CME propagation, expansion, and deformation with ANTEATR-PARADE (OSPREI's ANTEATR component, contained in PARADE.py), including drag toward the background solar wind and internal thermal/magnetic evolution, to predict arrival time at a satellite distance.

## When to use

Use when you have a ForeCAT far-corona terminal CME state (latitude, longitude, tilt, coronal speed) and need to propagate it through interplanetary space to predict arrival time, speed, size, and internal magnetic/thermal properties at a satellite.

## When NOT to use

- The CME has not yet been simulated through the corona and you need deflection/rotation from the solar surface — run ForeCAT first, since ANTEATR takes the ForeCAT terminal state as its initial condition
- You need synthetic in situ profiles at a spacecraft (magnetic field vector, velocity, Kp) — that is FIDO's role; ANTEATR only propagates the CME and gives bulk/arrival properties
- You only need a quick coronagraph-based arrival time estimate without expansion, deformation, or internal properties — simple drag-based or WSA-ENLIL+Cone arrival models may suffice

## Inputs

- ForeCAT terminal CME state (final latitude, longitude, tilt, coronal speed CMEvr) from the outer corona (~10 Rs)
- OSPREI input text file with REQUIRED fields date (YYYYMMDD), time (HH:MM), suffix, and models=ANT
- Satellite position parameters (SatLat, SatLon, SatR, e.g. 213 Rs for L1)
- Background solar wind parameters: SWCd, SWv, SWn, SWB

## Outputs

- Arrival time at the satellite distance (e.g. mean 18:28 UT 12 May 2021 for CME 1; 01:47 UT 25 Apr 2021 for CME 2)
- Final CME speed (propagation and expansion), size, and shape, plus profiles of these with distance
- Internal thermal and magnetic properties (mean durations 36 and 27 hr; mean densities 11.1 and 5.8 cm^-3; central B 18 vs 8 nT)
- Standardized output files processable with processOSPREI.py into a results object, histograms, and profiles

## How to apply

Prepare an OSPREI input text file with date (YYYYMMDD), time (HH:MM), suffix, and models=ANT so ANTEATR alone runs from the far-corona state, with satellite position defaults SatLat=0°, SatLon=0°, SatR=213 Rs. Load the ForeCAT terminal CME state into PARADE.py as the initial condition for interplanetary propagation. Run with drag toward the background solar wind (SWCd=1, SWv=400 km/s, SWn=5 cm^-3, SWB=6.9 nT — defaults corresponding to average OMNI values at 1 AU) and flux rope settings FRpol, FRB/FRT scaled from the CME speed, Gamma=1.33, and IVDf=0.5. Record the evolution with distance of propagation and expansion speeds, angular widths, aspect ratios, and internal magnetic field and temperature through to arrival at the satellite distance, then process standardized outputs with processOSPREI.py into a results object and histograms.

## Related tools

- **OSPREI** (Coupled Sun-to-Earth CME modeling suite; invoked with models=ANT to run ANTEATR alone from the far-corona state) — https://github.com/ckay314/OSPREI
- **ANTEATR-PARADE** (Interplanetary propagation model (fully contained in PARADE.py) with drag, expansion, and deformation; takes the ForeCAT CME and propagates it to the final satellite distance) — https://github.com/ckay314/OSPREI
- **ForeCAT** (Provides the terminal far-corona CME state (final latitude, longitude, tilt, coronal speed) that seeds ANTEATR-PARADE) — https://github.com/ckay314/OSPREI

## Examples

```
python OSPREI.py OSPtestcase.txt  # with date, time, suffix, models=ANT set in the input file; then python processOSPREI.py OSPtestcase.txt
```

## Evaluation signals

- The emitted CSV contains mean transit times matching reported values (~18:28 UT 12 May 2021 for CME 1; ~01:47 UT 25 Apr 2021 for CME 2), each falling within the reported ranges
- Mean durations (~36 hr for CME 1, ~27 hr for CME 2), mean densities (~11.1 and 5.8 cm^-3), and central B (18 vs 8 nT) each fall within the reported ranges
- Profiles of propagation/expansion speed, angular width, aspect ratio, internal B, and temperature are recorded monotonically through to the arrival time at the satellite distance (SatR=213 Rs)
- Output file names are generated from date and suffix and can be loaded by processOSPREI.py into a results object without errors
- For CME 1, the ensemble-mean arrival (12 May 2021 18:28 UT) should be compared against the observed magnetic obstacle arrival (22:00 UT) as a sanity check

## Limitations

- Background solar wind is the most under-developed component of the OSPREI suite; effects of different backgrounds on results are untested
- Only two CME cases (22 April and 09 May 2021) with similar coronal origins but different in situ properties have been used for validation; more cases are needed
- Parameter space is not optimized — the published results use real-time-analogous inputs rather than fine-tuned values, so best attainable results are unknown
- Slow, streamer blowout CMEs exhibit more motion at farther distances, so the default FCrmax=10 Rs hand-off should not be used for these cases

## Evidence

- [readme] ANTEATR takes the ForeCAT CME and propagates it to the final satellite distance. Like most arrival time models, ANTEATR includes a drag force that causes the CME speed to approach that of the background solar wind.: "ANTEATR takes the ForeCAT CME and propagates it to the final satellite distance. Like most arrival time models, ANTEATR includes a drag force that causes the CME speed to approach that of the"
- [readme] The output is the final CME speed (both propagation and expansion), size, and shape (and their profiles with distance) as well as the arrival time and internal thermal and magnetic properties of the CME.: "The output is the final CME speed (both propagation and expansion), size, and shape (and their profiles with distance) as well as the arrival time and internal thermal and magnetic properties of the"
- [methods] All solar wind properties have defaults corresponding to the average OMNI values at 1 AU: "All solar wind properties have defaults corresponding to the average OMNI values at 1 AU"
- [discussion] This positional information is then propagated to an arrival time model, ANTEATR.: "This positional information is then propagated to an arrival time model, ANTEATR."
- [readme] We default to 10 Rs for "normal" CMEs as little deflection or rotation tends to happen beyond this distances. Slow, streamer blowout CMEs exhibit more motion at farther distances so the default should not be used for these cases.: "We default to 10 Rs for "normal" CMEs as little deflection or rotation tends to happen beyond this distances. Slow, streamer blowout CMEs exhibit more motion at farther distances so the default"
