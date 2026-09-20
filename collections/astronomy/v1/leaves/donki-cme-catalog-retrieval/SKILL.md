---
name: donki-cme-catalog-retrieval
description: 'Use when configuring a CME simulation (e.g., OSPREI) from observational,
  real-time-analogous sources: you have a CME date and need catalog-derived latitude,
  longitude, tilt, coronal velocity, and angular width plus solar wind values.'
when_to_use_negative:
- The goal is a fine-tuned, best-fit scientific study of a CME rather than a real-time-analogous
  forecasting proof-of-concept (the paper explicitly avoids tuning inputs to achieve
  a best fit)
- The CME has no DONKI CMEAnalysis entry or usable coronagraph observations, so no
  catalog-derived geometry or kinematics exist to retrieve
- Only the PFSS magnetic background is needed and no CME input parameters are required
  (magnetogram/PFSS setup is a separate prerequisite, run once per day)
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: OSPREI
  role: Target simulation suite whose 34 input parameters (CMElat, CMElon, CMEtilt,
    CMEvr, CMEAW, CMEM, FRB, FRT, FRpol, SWv/SWn/SWB, etc.) are populated from the
    retrieved observational values
  repo: https://github.com/ckay314/OSPREI
- name: DONKI
  role: CCMC catalog providing CMEAnalysis entries with CME geometry and kinematics
    used as source-labeled inputs (e.g., entries 16870 and 16772)
- name: OMNI
  role: Database of upstream 1-AU solar wind properties supplying SWv, SWn, and SWB
    inputs; also the origin of the defaults (400 km/s, 5 cm^-3, 6.9 nT)
- name: JSOC HMI synchronic magnetogram
  role: Magnetogram source (hmi.Mrdailysynframe_720s) for the PFSS magnetic background
    required before running OSPREI
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1029/2021sw002914
    title: 'OSPREI: A Coupled Approach to Modeling CME‐Driven Space Weather With Automatically
      Generated, User‐Friendly Outputs'
  source_content_hash: sha256:a0e4463b6a06a884bde1ab8e4cb1d0bbd80186de880291fd20b70d86ad3d18b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ckay314/OSPREI
  tools:
  - DONKI
  - JSOC HMI synchronic magnetogram
  - OMNI
  - OSPREI
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# donki-cme-catalog-retrieval

## Summary

Retrieves CME geometry and kinematic parameter values from the DONKI CMEAnalysis catalog (kauai.ccmc.gsfc.nasa.gov) and complementary sources (GCS reconstructions, OMNI solar wind, HMI synchronic magnetograms) to populate real-time-analogous input parameters for OSPREI simulations of specific CMEs.

## When to use

Use when configuring a CME simulation (e.g., OSPREI) from observational, real-time-analogous sources: you have a CME date and need catalog-derived latitude, longitude, tilt, coronal velocity, and angular width plus solar wind values.

## When NOT to use

- The goal is a fine-tuned, best-fit scientific study of a CME rather than a real-time-analogous forecasting proof-of-concept (the paper explicitly avoids tuning inputs to achieve a best fit)
- The CME has no DONKI CMEAnalysis entry or usable coronagraph observations, so no catalog-derived geometry or kinematics exist to retrieve
- Only the PFSS magnetic background is needed and no CME input parameters are required (magnetogram/PFSS setup is a separate prerequisite, run once per day)

## Inputs

- DONKI CMEAnalysis catalog entries (kauai.ccmc.gsfc.nasa.gov, e.g., entries 16870 and 16772)
- GCS (Graduated Cylindrical Shell) reconstruction values for CME geometry
- OMNI database solar wind properties at 1 AU (speed, density, magnetic field strength)
- SDO/HMI synchronic magnetogram from the JSOC catalog (hmi.Mrdailysynframe_720s) for the eruption day
- SDO/AIA composite EUV imagery (sdo.gsfc.nasa.gov/data/aiahmi) for tilt estimation
- STEREO SECCHI coronagraph data (secchi.nrl.navy.mil/secchi_flight/images)

## Outputs

- Labeled 34-parameter OSPREI input table (per-parameter source labels: EUV, DONKI, GCS, OMNI, Default)
- OSPREI input text file with required NUMBER parameters (date, time, suffix, CMElat, CMElon, CMEtilt, CMEvr, CMEAW)
- DONKI CME event records with interplanetary shock associations (e.g., IPS 16784)

## How to apply

Locate the DONKI CMEAnalysis entry for the eruption of interest (e.g., entry 16870 for the 09 May 2021 CME and CME 16772 with interplanetary shock 16784 for the 22 April 2021 CME) at kauai.ccmc.gsfc.nasa.gov, using its reported values for CME geometry and kinematics. Pull upstream background solar wind speed, density, and magnetic field strength at 1 AU from the OMNI database (omniweb.gsfc.nasa.gov), falling back to the OMNI-average defaults (SWv 400 km/s, SWn 5 cm^-3, SWB 6.9 nT) where observations are unavailable. Check availability of an HMI synchronic magnetogram from the JSOC catalog for the day of the eruption (or the day before if the eruption precedes the magnetogram time) to build the PFSS background. For parameters without observational constraints (e.g., CMEtilt for CME 1 was visually estimated at 45 degrees from EUV observations; CME 2 used a 20-degree tilt; both cases used v_F = 800 km/s coronal velocities), label each of the 34 OSPREI input parameters with its source: EUV, DONKI, GCS, OMNI, or Default. The rationale is a forecasting proof-of-concept: use information analogous to what would be available in real time rather than fine-tuned best-fit values.

## Related tools

- **OSPREI** (Target simulation suite whose 34 input parameters (CMElat, CMElon, CMEtilt, CMEvr, CMEAW, CMEM, FRB, FRT, FRpol, SWv/SWn/SWB, etc.) are populated from the retrieved observational values) — https://github.com/ckay314/OSPREI
- **DONKI** (CCMC catalog providing CMEAnalysis entries with CME geometry and kinematics used as source-labeled inputs (e.g., entries 16870 and 16772))
- **OMNI** (Database of upstream 1-AU solar wind properties supplying SWv, SWn, and SWB inputs; also the origin of the defaults (400 km/s, 5 cm^-3, 6.9 nT))
- **JSOC HMI synchronic magnetogram** (Magnetogram source (hmi.Mrdailysynframe_720s) for the PFSS magnetic background required before running OSPREI)

## Examples

```
curl -s "https://kauai.ccmc.gsfc.nasa.gov/DONKI/view/CMEAnalysis/16870/1" -o donki_cme16870.html
```

## Evaluation signals

- The labeled input table contains exactly 34 data rows with the required fields (date, time, suffix, models, CMElat, CMElon, CMEtilt, CMEvr, CMEAW) and a per-parameter source-label column
- Spot values match the reference Table 1: CME 1 (09 May 2021) tilt = 45 degrees, CME 2 (22 April 2021) tilt = 20 degrees, both coronal velocities v_F = 800 km/s
- Every parameter carries a source label from the allowed set {EUV, DONKI, GCS, OMNI, Default}
- Solar wind defaults match OMNI averages at 1 AU over about a decade (SWv 400 km/s, SWn 5 cm^-3, SWB 6.9 nT) when OMNI observations are not used
- The HMI synchronic magnetogram date follows the rule: day of eruption, or day before if the eruption occurs before the magnetogram time

## Limitations

- CME tilt may not be catalog-derived: the 09 May 2021 tilt (45 degrees) was visually estimated from EUV observations, and some inputs fall back to defaults, so source labels must distinguish observed from assumed values
- The HMI synchronic magnetogram does not capture solar evolution rotating around near the eastern limb, and both CMEs had active regions in the east
- Solar wind properties are defaults from average OMNI values rather than event-specific measurements unless explicitly retrieved; the background solar wind is one of the most under-developed OSPREI components
- The real-time-analogous approach means results are not fine-tuned; OSPREI's best attainable results with optimized parameter-space exploration remain unknown

## Evidence

- [discussion] The DONKI entry for the 09 May 2021 CME is available at https://kauai.ccmc.gsfc.nasa.gov/DONKI/view/CMEAnalysis/16870/1: [evidence span withheld — anchor status: not_found]
- [discussion] OMNI data are available at https://omniweb.gsfc.nasa.gov/.: "OMNI data are available at https://omniweb.gsfc.nasa.gov/."
- [methods] All solar wind properties have defaults corresponding to the average OMNI values at 1 AU: "All solar wind properties have defaults corresponding to the average OMNI values at 1 AU"
- [readme] We recommend using an HMI synchronic magnetogram for the day of the eruption (or the day before if the eruption occurs before the time of the magnetogram). These are available from the JSOC catalog.: "We recommend using an HMI synchronic magnetogram for the day of the eruption (or the day before if the eruption occurs before the time of the magnetogram)."
- [abstract] using information analogous to what would be available in real time rather than fine-tuning input parameters to achieve a best fit: "using information analogous to what would be available in real time rather than fine-tuning input parameters to achieve a best fit"
