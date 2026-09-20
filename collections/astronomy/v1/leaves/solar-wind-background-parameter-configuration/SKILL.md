---
name: solar-wind-background-parameter-configuration
description: Use when setting up an OSPREI input text file for ANTEATR-PARADE propagation
  and you must specify the ambient solar wind state (drag coefficient, speed, density,
  magnetic field) that the CME propagates into.
when_to_use_negative:
- When the goal is FIDO in situ profile synthesis or ForeCAT coronal deflection rather
  than interplanetary arrival-time propagation — the solar wind terms belong to the
  ANTEATR (drag) portion of the chain
- 'When a time-varying or event-specific solar wind background is required: the article
  notes the background is ''one of the most under-developed components of the OSPREI
  suite'' and only static OMNI-average defaults exist'
- When using streamer-blowout or slow CME cases where ForeCAT defaults (e.g. FCrmax=10
  Rs) are inappropriate — but solar wind defaults still apply, so adjust CME parameters
  instead
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: OSPREI
  role: Coupled Sun-to-Earth CME suite whose input text file carries the SWCd/SWv/SWn/SWB
    background solar wind terms consumed by the ANTEATR drag model
  repo: https://github.com/ckay314/OSPREI
- name: ANTEATR (PARADE)
  role: Interplanetary propagation model (PARADE.py) that applies the drag force toward
    the configured background solar wind and computes arrival time and internal property
    evolution
  repo: https://github.com/ckay314/OSPREI
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1029/2021sw002914
    title: 'OSPREI: A Coupled Approach to Modeling CME‐Driven Space Weather With Automatically
      Generated, User‐Friendly Outputs'
  source_content_hash: sha256:3706a957c2cef19d7d55433669f5da4ad0a35de4307c5e407d160e988749df88
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ckay314/OSPREI
  tools:
  - ANTEATR
  - ANTEATR (PARADE)
  - ForeCAT
  - OSPREI
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# solar-wind-background-parameter-configuration

## Summary

Configures the background solar wind parameters (SWCd, SWv, SWn, SWB) that drive the drag and magnetic forces in OSPREI's ANTEATR-PARADE interplanetary propagation stage, using OMNI-averaged 1 AU defaults unless event-specific values are supplied.

## When to use

Use when setting up an OSPREI input text file for ANTEATR-PARADE propagation and you must specify the ambient solar wind state (drag coefficient, speed, density, magnetic field) that the CME propagates into.

## When NOT to use

- When the goal is FIDO in situ profile synthesis or ForeCAT coronal deflection rather than interplanetary arrival-time propagation — the solar wind terms belong to the ANTEATR (drag) portion of the chain
- When a time-varying or event-specific solar wind background is required: the article notes the background is 'one of the most under-developed components of the OSPREI suite' and only static OMNI-average defaults exist
- When using streamer-blowout or slow CME cases where ForeCAT defaults (e.g. FCrmax=10 Rs) are inappropriate — but solar wind defaults still apply, so adjust CME parameters instead

## Inputs

- OSPREI input text file (date, time, suffix, models=ANT, satellite position SatLat/SatLon/SatR)
- ForeCAT terminal CME state (final latitude, longitude, tilt, coronal speed) as initial condition for PARADE.py
- Optional event-specific solar wind measurements; otherwise OMNI 1 AU decade-averaged defaults (400 km/s, 5 cm^-3, 6.9 nT)

## Outputs

- Configured OSPREI input file with SWCd, SWv, SWn, SWB terms set for ANTEATR-PARADE
- ANTEATR output with CME transit/arrival time and profiles of propagation and expansion speeds, angular width, aspect ratio, internal B and temperature with distance

## How to apply

In the OSPREI input text file, set the background solar wind terms: SWCd=1 (drag coefficient), SWv=400 km/s, SWn=5 cm^-3, and SWB=6.9 nT, which correspond to average OMNI values at 1 AU; all solar wind properties have these defaults if omitted. These values determine the drag force that makes the CME speed approach the solar wind speed during ANTEATR-PARADE propagation — omitting drag (or using only distance/speed estimates) is precisely why simple arrival predictions err. For the Kay et al. (2022) validation cases, run ANTEATR alone (models=ANT) from the ForeCAT far-corona state toward a satellite at SatLat=0°, SatLon=0°, SatR=213 Rs, with flux rope settings FRpol, FRB/FRT scaled from the CME speed, Gamma=1.33, and IVDf=0.5 alongside the solar wind terms. The rationale for using OMNI averages rather than in situ measurements is the forecasting proof-of-concept design, where inputs are limited to real-time-analogous information rather than fine-tuned fits.

## Related tools

- **OSPREI** (Coupled Sun-to-Earth CME suite whose input text file carries the SWCd/SWv/SWn/SWB background solar wind terms consumed by the ANTEATR drag model) — https://github.com/ckay314/OSPREI
- **ANTEATR (PARADE)** (Interplanetary propagation model (PARADE.py) that applies the drag force toward the configured background solar wind and computes arrival time and internal property evolution) — https://github.com/ckay314/OSPREI

## Examples

```
SWCd=1; SWv=400; SWn=5; SWB=6.9  # solar wind terms in OSPREI input file, then: python OSPREI.py OSPtestcase.txt
```

## Evaluation signals

- Input file contains SWCd=1, SWv=400, SWn=5, SWB=6.9 (or documented OMNI-default equivalents) when solar wind terms are omitted
- Resulting ensemble-mean arrival times fall near 12 May 2021 18:28 UT (CME 1, observed 22:00 UT) and 25 April 2021 01:47 UT (CME 2), within reported ranges
- Mean durations of 36 hr and 27 hr and mean densities of 11.1 and 5.8 cm^-3 fall within the paper's reported ranges
- value_in_range check on central B (18 nT for CME 1 vs 8 nT for CME 2) confirms the drag-driven propagation did not unphysically alter internal properties
- CME speed approaches SWv at large distances rather than staying constant, confirming the drag term was active

## Limitations

- The background solar wind is 'one of the most under-developed components of the OSPREI suite and is an obvious area for improvement'; effects of different backgrounds are untested
- Defaults are static OMNI averages at 1 AU (~a decade), not event-specific or radial profiles of the actual ambient wind
- Simple distance/speed arrival estimates without drag 'lead to large errors as it neglects the change in CME speed due to drag' — the configured background must actually be used, not bypassed

## Evidence

- [methods] All solar wind properties have defaults corresponding to the average OMNI values at 1 AU: "All solar wind properties have defaults corresponding to the average OMNI values at 1 AU"
- [readme] ANTEATR drag toward background solar wind: "Like most arrival time models, ANTEATR includes a drag force that causes the CME speed to approach that of the background solar wind."
- [intro] Neglecting drag causes large arrival-time errors: "an arrival time estimated using simply the distance and speed, but this method leads to large errors as it neglects the change in CME speed due to drag"
- [discussion] Background solar wind is under-developed in OSPREI: [evidence span withheld — anchor status: not_found]
- [readme] Input-file-driven parameter assignment in OSPREI: "OSPREI is full specified using an text file where each input is assigned using specific terms."
