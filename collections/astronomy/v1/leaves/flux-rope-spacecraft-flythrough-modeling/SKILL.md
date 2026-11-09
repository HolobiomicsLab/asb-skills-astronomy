---
name: flux-rope-spacecraft-flythrough-modeling
description: Use when forecasting CME arrival and in situ magnetic field, velocity,
  or Kp profiles at Earth (or any satellite location) from a magnetogram- and coronagraph-constrained
  CME ensemble, and when verifying those predictions against OMNI observations.
when_to_use_negative:
- The goal is only an arrival time or speed estimate without magnetic structure —
  ANTEA alone or simpler drag-based arrival models suffice
- A real magnetometer time series already exists at the target location — compare
  against observations rather than flying a synthetic spacecraft
- The CME is not modeled as an internal magnetic flux rope (e.g., hydrodynamic pressure-pulse
  treatments), since FIDO depends on ForeCAT orientation and ANTEATR internal magnetic
  properties
tools:
- name: FIDO
  role: Passes the evolved CME over the synthetic spacecraft to determine the in situ
    magnetic field vector, velocity, and Kp index
  repo: https://github.com/ckay314/OSPREI
- name: OSPREI
  role: Coupled suite that invokes FIDO after ForeCAT and ANTEATR; input files, ensemble
    runs, and figure processing are driven through OSPREI.py and processOSPREI.py
  repo: https://github.com/ckay314/OSPREI
- name: ANTEATR
  role: Supplies the evolved CME (propagation, expansion, deformation, internal properties)
    that FIDO flies past the spacecraft
  repo: https://github.com/ckay314/OSPREI
- name: ForeCAT
  role: Provides the CME position and orientation used by FIDO to determine the spacecraft's
    path through the flux rope
  repo: https://github.com/ckay314/OSPREI
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1029/2021sw002914
    title: 'OSPREI: A Coupled Approach to Modeling CME‐Driven Space Weather With Automatically
      Generated, User‐Friendly Outputs'
  source_content_hash: sha256:81341b80cc281b4696ac3bbdfa9d4e0698c7c4eeaaba8532d9e91bfa1482542b
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# flux-rope-spacecraft-flythrough-modeling

## Summary

Flies an ensemble of ForeCAT/ANTEATR-evolved CME flux ropes over a synthetic spacecraft with FIDO (within OSPREI) to generate per-member in situ magnetic field, velocity, and Kp profiles, enabling prediction of maximum B, most negative Bz, and sheath Kp at Earth.

## When to use

Use when forecasting CME arrival and in situ magnetic field, velocity, or Kp profiles at Earth (or any satellite location) from a magnetogram- and coronagraph-constrained CME ensemble, and when verifying those predictions against OMNI observations.

## When NOT to use

- The goal is only an arrival time or speed estimate without magnetic structure — ANTEA alone or simpler drag-based arrival models suffice
- A real magnetometer time series already exists at the target location — compare against observations rather than flying a synthetic spacecraft
- The CME is not modeled as an internal magnetic flux rope (e.g., hydrodynamic pressure-pulse treatments), since FIDO depends on ForeCAT orientation and ANTEATR internal magnetic properties

## Inputs

- OSPREI input text file (.txt) with date, time, suffix, models, and satellite parameters (SatLat, SatLon, SatR, SatRot or .sat file)
- Optional .ens file specifying ensemble parameters and ranges (required when nRuns > 1)
- ANTEATR/PARADE evolved-CME outputs (final speed, size, shape, arrival time, internal thermal and magnetic properties)
- ForeCAT final CME position and orientation (latitude, longitude, tilt)
- PFSS background magnetic field pickles computed by harmonics.py and PFSS.py from an HMI synchronic magnetogram
- OMNI observational data file passed via ObsDataFile for overlay

## Outputs

- Per-ensemble-member in situ magnetic field (B, Bz) profiles at the synthetic spacecraft
- Per-member velocity profiles
- Kp index profiles computed from the in situ values
- In situ profile figures and ensemble probability contour plots from processOSPREI.py
- Verification table of maximum B, most negative Bz, arrival time, and maximum Kp

## How to apply

Configure an OSPREI input text file with date (YYYYMMDD), time (HH:MM), suffix, models set to the FIDO component (or ALL/IP), and satellite position parameters (SatLat, SatLon, SatR, SatRot or a .sat path file), setting includePUP=True to activate the FIDO-SIT sheath. Run FIDO over the ANTEATR ensemble outputs, where each ensemble member's evolved CME (position and orientation from ForeCAT) is passed over the synthetic spacecraft to yield per-member in situ B-field, velocity, and Kp profiles. Process standardized output files with processOSPREI.py into in situ profile figures and ensemble probability contour plots, overlaying OMNI observational data supplied via ObsDataFile for comparison. Judge correctness by requiring the ensemble-mean shock arrival within 3 hr of observation, maximum B of 16±3 nT (CME 1, 09 May 2021) and 11±1 nT (CME 2, 22 April 2021), most negative Bz of −11±2 nT and −2±1 nT respectively, and maximum Kp occurring in the sheath.

## Related tools

- **FIDO** (Passes the evolved CME over the synthetic spacecraft to determine the in situ magnetic field vector, velocity, and Kp index) — https://github.com/ckay314/OSPREI
- **OSPREI** (Coupled suite that invokes FIDO after ForeCAT and ANTEATR; input files, ensemble runs, and figure processing are driven through OSPREI.py and processOSPREI.py) — https://github.com/ckay314/OSPREI
- **ANTEATR** (Supplies the evolved CME (propagation, expansion, deformation, internal properties) that FIDO flies past the spacecraft) — https://github.com/ckay314/OSPREI
- **ForeCAT** (Provides the CME position and orientation used by FIDO to determine the spacecraft's path through the flux rope) — https://github.com/ckay314/OSPREI

## Examples

```
python OSPREI.py OSPtestcase.txt && python processOSPREI.py OSPtestcase.txt
```

## Evaluation signals

- Ensemble-mean predicted shock arrival time falls within 3 hr of the observed OMNI value for both CMEs
- Maximum B within the ensemble in situ profiles is 16±3 nT for CME 1 (09 May 2021) and 11±1 nT for CME 2 (22 April 2021)
- Most negative Bz is −11±2 nT (CME 1) and −2±1 nT (CME 2)
- The maximum Kp occurs in the sheath rather than the flux rope
- processOSPREI.py produces standardized in situ profile figures and ensemble probability contour plots from the results object without errors

## Limitations

- FIDO-SIT cannot predict in situ magnetic field components and can only assume a Parker Spiral orientation for the upstream direction
- The background solar wind is one of the most under-developed components of OSPREI and effects of different backgrounds are untested
- Results are demonstrated on only two CME cases; OSPREI better reproduces the in situ magnetic field of the more flux-rope-like CME, and whether this is a persistent limitation is unclear
- Inputs are not fine-tuned (forecasting proof-of-concept), so parameter space is unoptimized and best attainable results are unknown

## Evidence

- [intro] FIDO produces synthetic in situ profiles for the CME's interaction with a spacecraft: "Finally, ForeCAT In situ Data Observer produces in situ profiles for a CME's interaction with a synthetic spacecraft."
- [readme] README: FIDO flies the evolved CME over the synthetic spacecraft: "FIDO takes the evolved CME from ANTEATR with the position and orientation from ForeCAT and passes the CME over a synthetic spacecraft. The relative location of the spacecraft within the CME"
- [discussion] FIDO determines in situ magnetic field and velocity profile: [evidence span withheld — anchor status: not_found]
- [discussion] Parker Spiral orientation limitation: "the prediction of the actual components leaves much to be desired as we can only currently assume a Parker Spiral orientation for the upstream direction"
- [readme] Input file satellite and model configuration: "`models` - Selects whether to run all of OSPREI (ALL), the individual components (FC, ANT, FIDO), or some combination (IP - ANT+FIDO, noB - FC+ANT)."
