---
name: sheath-inclusion-in-synthetic-insitu-profiles
description: Use when producing ensemble in situ predictions for CME arrival at a
  synthetic spacecraft and the validation requires the maximum Kp to occur in the
  sheath, or when comparing predicted B and Bz extrema against OMNI observations that
  include sheath structure.
when_to_use_negative:
- When only flux rope (cloud) interior structure is of interest and sheath contamination
  would confound the comparison — includePUP should not be set
- When the goal requires predicting the actual in situ magnetic field component directions,
  since FIDO-SIT assumes only a Parker Spiral orientation for the upstream direction
- When the target is background solar wind modeling rather than CME-driven structure
  — the background solar wind is one of the most under-developed OSPREI components
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: FIDO
  role: ForeCAT In situ Data Observer — flies the evolved CME over a synthetic spacecraft
    to produce in situ magnetic field, velocity, and Kp profiles; includePUP=True
    activates the FIDO-SIT sheath
  repo: https://github.com/ckay314/OSPREI
- name: OSPREI
  role: Main wrapper invoked with an input text file; its results object feeds processOSPREI.py
    for profile figures and the verification table
  repo: https://github.com/ckay314/OSPREI
- name: ANTEATR
  role: Supplies the evolved CME (propagation, expansion, deformation, internal properties)
    that FIDO flies past the synthetic spacecraft
  repo: https://github.com/ckay314/OSPREI
- name: ForeCAT
  role: Provides the CME position and orientation used when FIDO passes the CME over
    the spacecraft
  repo: https://github.com/ckay314/OSPREI
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1029/2021sw002914
    title: 'OSPREI: A Coupled Approach to Modeling CME‐Driven Space Weather With Automatically
      Generated, User‐Friendly Outputs'
  source_content_hash: sha256:e24285572068ce7cdcb1353b326230f6126cfd7045bf0f9dd3a94fd5d3ec3f90
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ckay314/OSPREI
  tools:
  - ANTEA
  - ANTEATR
  - FIDO
  - ForeCAT
  - OSPREI
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# Sheath inclusion in synthetic in situ profiles

## Summary

Activates the FIDO-SIT sheath (includePUP=True) when running FIDO within OSPREI so that synthetic in situ magnetic field, velocity, and Kp profiles capture the turbulent sheath ahead of the flux rope, not just the flux rope itself.

## When to use

Use when producing ensemble in situ predictions for CME arrival at a synthetic spacecraft and the validation requires the maximum Kp to occur in the sheath, or when comparing predicted B and Bz extrema against OMNI observations that include sheath structure.

## When NOT to use

- When only flux rope (cloud) interior structure is of interest and sheath contamination would confound the comparison — includePUP should not be set
- When the goal requires predicting the actual in situ magnetic field component directions, since FIDO-SIT assumes only a Parker Spiral orientation for the upstream direction
- When the target is background solar wind modeling rather than CME-driven structure — the background solar wind is one of the most under-developed OSPREI components

## Inputs

- OSPREI input text file with date, time, suffix, models=FIDO, satellite parameters (SatLat, SatLon, SatR, SatRot or .sat path file), and includePUP=True
- ANTEATR (PARADE) ensemble outputs containing each ensemble member's evolved CME speed, size, shape, and internal thermal and magnetic properties
- ForeCAT positional/orientational outputs (final latitude, longitude, tilt and their radial profiles)
- OMNI observational in situ data supplied via ObsDataFile for overlay

## Outputs

- Per-ensemble-member synthetic in situ magnetic field, velocity, and Kp profiles at the spacecraft
- In situ profile figures and ensemble probability contour plots from processOSPREI.py
- Verification table reporting maximum B (16±3 nT CME 1; 11±1 nT CME 2), most negative Bz (−11±2 nT; −2±1 nT), shock arrival time, and sheath Kp maximum

## How to apply

In the OSPREI input text file for each CME (date, time, suffix set; models set to the FIDO component), set satellite position parameters (SatLat, SatLon, SatR, SatRot or a .sat path file) and set includePUP=True to activate the FIDO-SIT sheath. FIDO then takes the evolved CME from ANTEATR — with position and orientation from ForeCAT — and flies it over the synthetic spacecraft, where the relative spacecraft location determines the in situ magnetic field vector and velocity; the sheath component contributes the upstream, compressed solar wind region ahead of the flux rope. Run FIDO over each ANTEATR ensemble member to get per-member profiles, then process with processOSPREI.py to build in situ profile figures, ensemble probability contours, and the verification table with OMNI data overlaid via ObsDataFile. Correct application is judged by: ensemble-mean shock arrival within 3 hr of observed; maximum B of 16±3 nT (CME 1, 09 May 2021) and 11±1 nT (CME 2, 22 April 2021); most negative Bz of −11±2 nT and −2±1 nT respectively; and maximum Kp occurring in the sheath rather than the flux rope.

## Related tools

- **FIDO** (ForeCAT In situ Data Observer — flies the evolved CME over a synthetic spacecraft to produce in situ magnetic field, velocity, and Kp profiles; includePUP=True activates the FIDO-SIT sheath) — https://github.com/ckay314/OSPREI
- **OSPREI** (Main wrapper invoked with an input text file; its results object feeds processOSPREI.py for profile figures and the verification table) — https://github.com/ckay314/OSPREI
- **ANTEATR** (Supplies the evolved CME (propagation, expansion, deformation, internal properties) that FIDO flies past the synthetic spacecraft) — https://github.com/ckay314/OSPREI
- **ForeCAT** (Provides the CME position and orientation used when FIDO passes the CME over the spacecraft) — https://github.com/ckay314/OSPREI

## Examples

```
python OSPREI.py OSPtestcase.txt  # input file sets models=FIDO, SatLat/SatLon/SatR/SatRot, includePUP=True; then: python processOSPREI.py OSPtestcase.txt
```

## Evaluation signals

- Maximum magnetic field within the ensemble in situ profiles equals 16±3 nT for CME 1 and 11±1 nT for CME 2
- Most negative Bz equals −11±2 nT for CME 1 and −2±1 nT for CME 2
- Ensemble-mean predicted shock arrival time falls within 3 hr of the observed value for both CMEs
- The maximum Kp in each synthetic profile occurs in the sheath, not in the flux rope
- Synthetic profiles successfully overlay OMNI observations for both CMEs when processed with processOSPREI.py

## Limitations

- FIDO-SIT cannot predict in situ magnetic field components; it assumes only a Parker Spiral orientation for the upstream direction
- Only two CME cases were considered; whether OSPREI's in situ magnetic field reproduction (better for the more flux-rope-like CME) generalizes is unresolved
- The background solar wind is one of the most under-developed OSPREI components and affects the sheath/solar wind structure in the profiles

## Evidence

- [intro] FIDO produces synthetic in situ profiles for the CME's interaction with a spacecraft: "Finally, ForeCAT In situ Data Observer produces in situ profiles for a CME's interaction with a synthetic spacecraft."
- [discussion] FIDO determines in situ magnetic field and velocity profile at a synthetic spacecraft: [evidence span withheld — anchor status: not_found]
- [readme] FIDO component behavior in README: "FIDO takes the evolved CME from ANTEATR with the position and orientation from ForeCAT and passes the CME over a synthetic spacecraft."
- [readme] Kp index computation in README: "It also calculates the Kp index from these values."
