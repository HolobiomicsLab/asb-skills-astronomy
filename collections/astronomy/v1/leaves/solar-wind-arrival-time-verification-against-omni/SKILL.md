---
name: solar-wind-arrival-time-verification-against-omni
description: Use when ensemble CME in situ predictions (B, Bz, velocity, Kp profiles
  from FIDO/OSPREI) must be validated against observed solar wind data such as OMNI
  at 1 AU, including shock arrival time within a tolerance.
when_to_use_negative:
- When only an arrival time is needed without in situ properties — a simpler arrival
  time model (e.g., ANTEATR alone or WSA-ENLIL + Cone) suffices
- When no observational in situ data (e.g., OMNI) are available for the event, since
  the overlay and threshold comparison cannot be performed
- When the goal is predicting in situ magnetic field components in the sheath — FIDO-SIT
  only assumes a Parker Spiral orientation for the upstream direction
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3998
tools:
- name: OSPREI
  role: Coupled suite invoked via OSPREI.py with an input text file; results processed
    into figures and verification table via processOSPREI.py
  repo: https://github.com/ckay314/OSPREI
- name: FIDO
  role: Flies each evolved CME over a synthetic spacecraft to produce in situ magnetic
    field, velocity, and Kp profiles; includePUP=True activates the FIDO-SIT sheath
  repo: https://github.com/ckay314/OSPREI
- name: ANTEATR
  role: Supplies the interplanetary propagation ensemble (expansion, deformation,
    arrival time) on which FIDO runs
  repo: https://github.com/ckay314/OSPREI
- name: ForeCAT
  role: Provides CME position and orientation used by FIDO for the in situ profile
    calculation
  repo: https://github.com/ckay314/OSPREI
- name: OMNI data
  role: Observational solar wind records overlaid on the synthetic profiles via ObsDataFile
    for verification
  repo: https://omniweb.gsfc.nasa.gov/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1029/2021sw002914
    title: 'OSPREI: A Coupled Approach to Modeling CME‐Driven Space Weather With Automatically
      Generated, User‐Friendly Outputs'
  source_content_hash: sha256:921fe774468e36537680a26f65a97f15570428cd82fd7f8834e657162b9198ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ckay314/OSPREI
  tools:
  - ANTEA
  - ANTEATR
  - FIDO
  - ForeCAT
  - OMNI data
  - OSPREI
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3998
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# solar-wind-arrival-time-verification-against-omni

## Summary

Verification workflow for OSPREI/FIDO ensemble in situ predictions: run FIDO over ANTEATR-evolved CMEs at a synthetic spacecraft, overlay OMNI observations via ObsDataFile, and check arrival time, magnetic field strength, Bz, and Kp against quantitative acceptance thresholds.

## When to use

Use when ensemble CME in situ predictions (B, Bz, velocity, Kp profiles from FIDO/OSPREI) must be validated against observed solar wind data such as OMNI at 1 AU, including shock arrival time within a tolerance.

## When NOT to use

- When only an arrival time is needed without in situ properties — a simpler arrival time model (e.g., ANTEATR alone or WSA-ENLIL + Cone) suffices
- When no observational in situ data (e.g., OMNI) are available for the event, since the overlay and threshold comparison cannot be performed
- When the goal is predicting in situ magnetic field components in the sheath — FIDO-SIT only assumes a Parker Spiral orientation for the upstream direction

## Inputs

- OSPREI input text files with date, time, suffix, models (FIDO component), satellite parameters (SatLat, SatLon, SatR, SatRot or .sat path file), and includePUP=True
- ANTEATR ensemble outputs (evolved CME position, size, shape, and internal thermal/magnetic properties)
- ForeCAT CME position and orientation outputs
- OMNI observational solar wind data supplied via the ObsDataFile parameter (from https://omniweb.gsfc.nasa.gov/)
- OSPREI standardized output files from the ensemble run

## Outputs

- Per-member in situ magnetic field, velocity, and Kp profiles at the synthetic spacecraft
- In situ profile figures with overlaid OMNI observations and ensemble probability contour plots
- Verification table assembled from the processOSPREI.py results object (arrival time, max B, most negative Bz, max Kp and sheath location)

## How to apply

Configure OSPREI input text files for each CME with date (YYYYMMDD), time (HH:MM), suffix, models set to the FIDO component, satellite position parameters (SatLat, SatLon, SatR, SatRot or a .sat path file), and includePUP=True to activate the FIDO-SIT sheath. Run FIDO on the ANTEATR ensemble outputs so each ensemble member's evolved CME (position and orientation from ForeCAT) passes over the synthetic spacecraft, producing per-member in situ magnetic field, velocity, and Kp profiles. Process the standardized output files with processOSPREI.py to create in situ profile figures and ensemble probability contour plots (contour computation can be skipped for quick-look runs). Overlay OMNI observations via the ObsDataFile input onto the synthetic profiles, then assemble the verification table from the results object. Validate that the ensemble-mean predicted shock arrival time falls within 3 hr of the observed value, maximum B matches 16±3 nT (CME 1, 09 May 2021) and 11±1 nT (CME 2, 22 April 2021), most negative Bz matches −11±2 nT and −2±1 nT respectively, and that the maximum Kp occurs in the sheath.

## Related tools

- **OSPREI** (Coupled suite invoked via OSPREI.py with an input text file; results processed into figures and verification table via processOSPREI.py) — https://github.com/ckay314/OSPREI
- **FIDO** (Flies each evolved CME over a synthetic spacecraft to produce in situ magnetic field, velocity, and Kp profiles; includePUP=True activates the FIDO-SIT sheath) — https://github.com/ckay314/OSPREI
- **ANTEATR** (Supplies the interplanetary propagation ensemble (expansion, deformation, arrival time) on which FIDO runs) — https://github.com/ckay314/OSPREI
- **ForeCAT** (Provides CME position and orientation used by FIDO for the in situ profile calculation) — https://github.com/ckay314/OSPREI
- **OMNI data** (Observational solar wind records overlaid on the synthetic profiles via ObsDataFile for verification) — https://omniweb.gsfc.nasa.gov/

## Examples

```
python OSPREI.py OSPtestcase.txt && python processOSPREI.py OSPtestcase.txt
```

## Evaluation signals

- Ensemble-mean predicted shock arrival time falls within 3 hr of the observed arrival time for both CMEs
- Ensemble maximum magnetic field is 16±3 nT for CME 1 (09 May 2021) and 11±1 nT for CME 2 (22 April 2021)
- Most negative Bz is −11±2 nT (CME 1) and −2±1 nT (CME 2) within the ensemble in situ profiles
- The maximum Kp occurs in the sheath (FIDO-SIT region activated with includePUP=True)
- Synthetic profiles and OMNI observations align on the same time axis in the overlaid profile figures

## Limitations

- FIDO-SIT cannot predict in situ magnetic field components; it only assumes a Parker Spiral orientation for the upstream direction
- Only two CME cases were considered; OSPREI better reproduces the in situ magnetic field of the more flux-rope-like CME and it is unclear whether this is a persistent limitation
- Inputs were not fine-tuned (real-time-analogous information), so results do not represent the best attainable; optimizing requires careful exploration of parameter space
- The background solar wind is one of the most under-developed components of OSPREI and effects of different backgrounds are untested

## Evidence

- [other] ensemble-mean predicted shock arrival time must fall within 3 hr of the observed value: [evidence span withheld — anchor status: not_found]
- [readme] FIDO synthetic spacecraft profiles: "FIDO takes the evolved CME from ANTEATR with the position and orientation from ForeCAT and passes the CME over a synthetic spacecraft."
- [discussion] OMNI overlay: "OMNI data are available at https://omniweb.gsfc.nasa.gov/."
- [readme] processOSPREI results object: "This takes the standardized output files, processes them into a results object, then makes an assortment of figures depending on whether a single simulation or a full ensemble was run."
- [discussion] FIDO-SIT Parker Spiral limitation: "the prediction of the actual components leaves much to be desired as we can only currently assume a Parker Spiral orientation for the upstream direction"
