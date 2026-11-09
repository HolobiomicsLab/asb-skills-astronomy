---
name: omni-solar-wind-data-extraction
description: Use when configuring Sun-to-Earth CME model inputs (e.g., OSPREI ANTEATR/FIDO
  runs) that require background solar wind speed, density, or magnetic field strength
  at the satellite location, and real-time-analogous 1 AU values rather than fine-tuned
  defaults are needed.
when_to_use_negative:
- The needed parameters are CME geometry or kinematics (CMElat, CMElon, CMEtilt, CMEvr,
  CMEAW) — these come from EUV/DONKI/GCS sources, not OMNI
- The analysis needs the coronal magnetic background (PFSS) — that requires an HMI
  synchronic magnetogram from the JSOC catalog, not OMNI solar wind data
- Only default climatological solar wind conditions are acceptable, in which case
  OSPREI's built-in OMNI-average defaults (400 km/s, 5 cm^-3, 6.9 nT) can be used
  without extraction
edam_operation: http://edamontology.org/operation_2422
tools:
- name: OMNI
  role: Source database for 1 AU background solar wind speed, density, and magnetic
    field strength used to set SWv, SWn, SWB
  repo: https://omniweb.gsfc.nasa.gov/
- name: OSPREI
  role: Consumer of the extracted OMNI values as SWv/SWn/SWB inputs; also ships OMNI-average
    defaults for these parameters
  repo: https://github.com/ckay314/OSPREI
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1029/2021sw002914
    title: 'OSPREI: A Coupled Approach to Modeling CME‐Driven Space Weather With Automatically
      Generated, User‐Friendly Outputs'
  source_content_hash: sha256:10d9c9890207cdb55c21317282a9de6e8c53aa3df6188458dfee2376cec26b62
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OMNI
  - OSPREI
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# omni-solar-wind-data-extraction

## Summary

Retrieves upstream background solar wind properties (speed, density, magnetic field strength) at 1 AU from the OMNI database (omniweb.gsfc.nasa.gov) to populate the SWv, SWn, and SWB input parameters of an OSPREI simulation, replacing or validating the OMNI-average defaults.

## When to use

Use when configuring Sun-to-Earth CME model inputs (e.g., OSPREI ANTEATR/FIDO runs) that require background solar wind speed, density, or magnetic field strength at the satellite location, and real-time-analogous 1 AU values rather than fine-tuned defaults are needed.

## When NOT to use

- The needed parameters are CME geometry or kinematics (CMElat, CMElon, CMEtilt, CMEvr, CMEAW) — these come from EUV/DONKI/GCS sources, not OMNI
- The analysis needs the coronal magnetic background (PFSS) — that requires an HMI synchronic magnetogram from the JSOC catalog, not OMNI solar wind data
- Only default climatological solar wind conditions are acceptable, in which case OSPREI's built-in OMNI-average defaults (400 km/s, 5 cm^-3, 6.9 nT) can be used without extraction

## Inputs

- OMNI database records at 1 AU (https://omniweb.gsfc.nasa.gov/) for solar wind speed, density, and magnetic field strength around the eruption date
- OSPREI input text file with SWv, SWn, SWB parameter slots (km/s, cm^-3, nT)
- CME eruption date/time (YYYYMMDD, HH:MM) to define the OMNI query window

## Outputs

- Background solar wind parameter set (SWv, SWn, SWB) with per-parameter source label 'OMNI'
- Rows in the labeled OSPREI 34-parameter input table (e.g., osprei_input_table.csv)

## How to apply

Query the OMNI database at https://omniweb.gsfc.nasa.gov/ for the upstream solar wind speed, density, and magnetic field strength at 1 AU covering the time of the CME eruption (e.g., the 22 April and 09 May 2021 events). Record these as the SWv (km/s), SWn (cm^-3), and SWB (nT) values in the OSPREI input text file, and label each parameter's provenance as 'OMNI' in the per-parameter source table. Note that if no values are supplied, OSPREI falls back to defaults corresponding to the average OMNI values at 1 AU over about a decade (SWv 400 km/s, SWn 5 cm^-3, SWB 6.9 nT), so explicitly entered OMNI values override these climatological defaults. Use these solar wind values as the drag-bath conditions for ANTEATR propagation and as the upstream state for FIDO's synthetic in situ profiles.

## Related tools

- **OMNI** (Source database for 1 AU background solar wind speed, density, and magnetic field strength used to set SWv, SWn, SWB) — https://omniweb.gsfc.nasa.gov/
- **OSPREI** (Consumer of the extracted OMNI values as SWv/SWn/SWB inputs; also ships OMNI-average defaults for these parameters) — https://github.com/ckay314/OSPREI

## Evaluation signals

- The extracted SWv, SWn, SWB values are labeled with source 'OMNI' in the input parameter table
- Explicit OMNI values differ from (or are validated against) OSPREI's built-in OMNI-average defaults of 400 km/s, 5 cm^-3, and 6.9 nT
- The OSPREI input file parses and the ANTEATR/FIDO run completes with the supplied solar wind values (drag bath and upstream state accepted)
- The query window matches the eruption date/time of the target CME (e.g., 09 May or 22 April 2021) so upstream conditions are time-appropriate

## Limitations

- OSPREI's default solar wind values are decadal OMNI averages, and the background solar wind is described as one of the most under-developed components of the OSPREI suite and an obvious area for improvement
- OMNI provides the background field strength only; FIDO-SIT currently assumes a Parker Spiral orientation for the upstream direction rather than predicting actual magnetic field components
- Effects of using different background solar winds on OSPREI results had not yet been tested

## Evidence

- [methods] All solar wind properties have defaults corresponding to the average OMNI values at 1 AU: "All solar wind properties have defaults corresponding to the average OMNI values at 1 AU"
- [discussion] OMNI data are available at https://omniweb.gsfc.nasa.gov/.: "OMNI data are available at https://omniweb.gsfc.nasa.gov/."
- [readme] SWB - The background solar wind magnetic field strength at the location of the satellite.: "SWB - The background solar wind magnetic field strength at the location of the satellite."
- [readme] SWv - The background solar wind speed at the location of the satellite.: "SWv - The background solar wind speed at the location of the satellite."
