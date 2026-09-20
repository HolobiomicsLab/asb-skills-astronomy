---
name: cme-parameter-source-attribution
description: Use when configuring OSPREI (or similar CME forecast-model) input files
  for specific CME events and you must document where every parameter value comes
  from — e.g., building a labeled parameter table from DONKI CMEAnalysis entries,
  GCS reconstructions, OMNI solar wind, and EUV imagery.
when_to_use_negative:
- When performing scientific-parameter-space optimization or best-fit tuning of OSPREI
  inputs — this skill deliberately uses real-time-analogous information rather than
  fine-tuned inputs, which the article notes leaves best attainable results unknown
- When only an arrival-time estimate from distance/speed is needed — that simple method
  neglects CME–solar-wind drag and is what OSPREI's coupled ForeCAT/ANTEATR/FIDO chain
  is meant to replace
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: OSPREI (Open Solar Physics Rapid Ensemble Information)
  role: Target model whose 34 input parameters (date, time, suffix, models, CMElat/lon/tilt/vr/AW,
    CMEM, FRB, FRT, FRpol, SWv/SWn/SWB, satellite parameters) are being sourced and
    labeled
  repo: https://github.com/ckay314/OSPREI
- name: DONKI
  role: Catalog of CMEAnalysis entries supplying CME kinematic parameters (e.g., coronal
    velocities) for the 09 May and 22 April 2021 CMEs
- name: OMNI
  role: Source of background solar wind speed, density, and magnetic field strength
    at 1 AU; also the basis of OSPREI's solar wind defaults
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1029/2021sw002914
    title: 'OSPREI: A Coupled Approach to Modeling CME‐Driven Space Weather With Automatically
      Generated, User‐Friendly Outputs'
  source_content_hash: sha256:171e863fb8ddb60e4bd6beda08b830f09e7c5ab3a54759f79c81fdc283385a20
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ckay314/OSPREI
  tools:
  - DONKI
  - OMNI
  - OSPREI (Open Solar Physics Rapid Ensemble Information)
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cme-parameter-source-attribution

## Summary

Populates a complete OSPREI input table (e.g., 34 parameters such as CMElat, CMElon, CMEtilt, CMEvr, CMEAW, CMEM, FRB, SWv/SWn/SWB) for forecasting-analog CME cases, assigning each parameter a source label (EUV, DONKI, GCS, OMNI, or Default) so inputs mimic real-time availability rather than fine-tuned best fits.

## When to use

Use when configuring OSPREI (or similar CME forecast-model) input files for specific CME events and you must document where every parameter value comes from — e.g., building a labeled parameter table from DONKI CMEAnalysis entries, GCS reconstructions, OMNI solar wind, and EUV imagery.

## When NOT to use

- When performing scientific-parameter-space optimization or best-fit tuning of OSPREI inputs — this skill deliberately uses real-time-analogous information rather than fine-tuned inputs, which the article notes leaves best attainable results unknown
- When only an arrival-time estimate from distance/speed is needed — that simple method neglects CME–solar-wind drag and is what OSPREI's coupled ForeCAT/ANTEATR/FIDO chain is meant to replace

## Inputs

- DONKI CMEAnalysis catalog entries (kauai.ccmc.gsfc.nasa.gov), e.g., entry 16870 (09 May 2021 CME) and entry 16772 with interplanetary shock 16784 (22 April 2021 CME)
- OMNI database solar wind properties at 1 AU (speed, density, magnetic field strength) from omniweb.gsfc.nasa.gov
- GCS (GCS model) CME geometry/kinematics reconstructions
- EUV observations (SDO composite imagery) for visual estimates such as CME tilt
- HMI synchronic magnetogram availability check via JSOC catalog (hmi.Mrdailysynframe_720s) for the eruption day

## Outputs

- Labeled OSPREI input parameter table (e.g., osprei_input_table.csv) with 34 data rows, required fields (date, time, suffix, models, CMElat, CMElon, CMEtilt, CMEvr, CMEAW), and a per-parameter source-label column (EUV, DONKI, GCS, OMNI, Default)

## How to apply

Retrieve the DONKI CMEAnalysis entries for each CME (e.g., entry 16870 for the 09 May 2021 CME; entry 16772 with IPS 16784 for the 22 April 2021 CME) from kauai.ccmc.gsfc.nasa.gov, and pull upstream solar wind speed, density, and magnetic field strength at 1 AU from the OMNI database. Check JSOC for the HMI synchronic magnetogram for the day of each eruption (day before if the eruption precedes the magnetogram time). For each OSPREI input parameter — date, time, suffix, models, CMElat/lon/tilt/vr/AW, CMEAWp, CMEM, CMEdelAx, CMEdelCS, FCrmax, FRB, FRT, FRpol, SWv/SWn/SWB, satellite parameters — assign a source label: DONKI/GCS values for CME geometry and kinematics (e.g., CME 1 tilt 45° estimated visually from EUV observations, CME 2 tilt 20°, both coronal velocities v_F = 800 km/s), OMNI values for solar wind (defaults correspond to average OMNI values at 1 AU), EUV for visually estimated quantities like tilt, and 'Default' where README defaults apply (e.g., CMEdelAx 0.75, CMEdelCS 1, SWCd 1). Write the labeled table (e.g., osprei_input_table.csv) and verify it contains exactly 34 data rows with required fields (date, time, suffix, models, CMElat, CMElon, CMEtilt, CMEvr, CMEAW) plus the source-label column, and that spot values match the reference Table 1. The rationale is a forecasting proof-of-concept: use information analogous to real-time availability rather than fine-tuning inputs to achieve a best fit.

## Related tools

- **OSPREI (Open Solar Physics Rapid Ensemble Information)** (Target model whose 34 input parameters (date, time, suffix, models, CMElat/lon/tilt/vr/AW, CMEM, FRB, FRT, FRpol, SWv/SWn/SWB, satellite parameters) are being sourced and labeled) — https://github.com/ckay314/OSPREI
- **DONKI** (Catalog of CMEAnalysis entries supplying CME kinematic parameters (e.g., coronal velocities) for the 09 May and 22 April 2021 CMEs)
- **OMNI** (Source of background solar wind speed, density, and magnetic field strength at 1 AU; also the basis of OSPREI's solar wind defaults)

## Examples

```
python OSPREI.py OSPtestcase.txt
```

## Evaluation signals

- CSV contains exactly 34 data rows
- Required fields present: date, time, suffix, models, CMElat, CMElon, CMEtilt, CMEvr, CMEAW, plus a source-label column
- Spot values match reference Table 1: CME 1 tilt 45°, CME 2 tilt 20°, both coronal velocities v_F = 800 km/s
- Every parameter carries one of the allowed source labels: EUV, DONKI, GCS, OMNI, or Default
- Solar wind entries (SWv, SWn, SWB) trace to OMNI 1 AU values rather than fine-tuned values

## Limitations

- Only two CME cases (22 April and 09 May 2021) were considered; more cases are needed to establish the approach
- Inputs are not fine-tuned or optimized — the parameter space has not been explored, so best attainable results are unknown
- The synchronic magnetogram does not capture solar evolution rotating around near the eastern limb; for both CMEs, active regions appear in the east and the effect is unquantified
- Some parameters are effectively unobservable — e.g., CMEdelAx (aspect ratio of the CME's central axis) 'is currently hard if not impossible to get a observationally measure', so values are guesses or defaults

## Evidence

- [abstract] Forecasting proof-of-concept approach using information analogous to real-time availability rather than fine-tuning inputs: "using information analogous to what would be available in real time rather than fine‐tuning input parameters to achieve a best fit"
- [discussion] DONKI entry for the 09 May 2021 CME is hosted at kauai.ccmc.gsfc.nasa.gov: [evidence span withheld — anchor status: not_found]
- [methods] Solar wind defaults correspond to average OMNI values at 1 AU: "All solar wind properties have defaults corresponding to the average OMNI values at 1 AU"
- [methods] HMI synchronic magnetogram timing rule for eruption day: "We recommend using an HMI synchronic magnetogram for the day of the eruption"
- [readme] README defines required OSPREI parameters and defaults used for source labeling: "`CMEtilt` The initial tilt or orientation of the CME, measured in degrees counterclockwise from solar west. *[REQUIRED]*"
