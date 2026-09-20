---
name: hmi-magnetogram-availability-checking
description: Use when configuring an OSPREI/ForeCAT simulation and you must confirm
  a usable HMI synchronic magnetogram exists in the JSOC catalog for the eruption
  date, applying the day-before fallback rule when the eruption occurs before the
  magnetogram time.
when_to_use_negative:
- You need upstream solar wind properties (speed, density, magnetic field at 1 AU)
  — those come from the OMNI database, not HMI magnetograms
- You need CME geometry and kinematics (tilt, velocity, angular width) — those come
  from DONKI/GCS/EUV observations, not magnetogram availability checks
- A magnetogram has already been downloaded and the PFSS pickle background computed
  — this skill is only the availability gate, not harmonics.py/PFSS.py execution
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: JSOC catalog
  role: Source catalog queried to confirm HMI synchronic magnetogram availability
    for the eruption day
- name: SDO/HMI magnetograms (hmi.Mrdailysynframe_720s)
  role: The specific magnetogram data series whose availability is checked and which
    feeds the PFSS magnetic background
- name: OSPREI
  role: Downstream simulation suite whose ForeCAT component requires the PFSS background
    built from the confirmed magnetogram
  repo: https://github.com/ckay314/OSPREI
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1029/2021sw002914
    title: 'OSPREI: A Coupled Approach to Modeling CME‐Driven Space Weather With Automatically
      Generated, User‐Friendly Outputs'
  source_content_hash: sha256:a982fef004b864205e4aea232d48c2cf017859d678c0f49a318c45bf3f2caca0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ckay314/OSPREI
  tools:
  - JSOC catalog
  - OSPREI
  - SDO/HMI magnetograms (hmi.Mrdailysynframe_720s)
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# hmi-magnetogram-availability-checking

## Summary

Checks the JSOC catalog for an SDO/HMI synchronic magnetogram covering the day of each CME eruption (or the day before if the eruption occurs before the magnetogram time), so the PFSS magnetic background required by OSPREI/ForeCAT can be built. This availability gate comes before downloading, Carrington-shifting (sync2synop.py), and harmonic-coefficient computation (harmonics.py).

## When to use

Use when configuring an OSPREI/ForeCAT simulation and you must confirm a usable HMI synchronic magnetogram exists in the JSOC catalog for the eruption date, applying the day-before fallback rule when the eruption occurs before the magnetogram time.

## When NOT to use

- You need upstream solar wind properties (speed, density, magnetic field at 1 AU) — those come from the OMNI database, not HMI magnetograms
- You need CME geometry and kinematics (tilt, velocity, angular width) — those come from DONKI/GCS/EUV observations, not magnetogram availability checks
- A magnetogram has already been downloaded and the PFSS pickle background computed — this skill is only the availability gate, not harmonics.py/PFSS.py execution

## Inputs

- CME eruption dates (YYYYMMDD) and eruption start times (HH:MM) from DONKI CMEAnalysis entries
- JSOC catalog hmi.Mrdailysynframe_720s HMI synchronic magnetogram series

## Outputs

- Per-CME availability decision: magnetogram date selected (day of eruption or day before)
- Identified fits file for download, to be saved as HMIYYYYMMDDsync.fits for the PFSS background pipeline

## How to apply

For each CME eruption date, query the JSOC catalog for the hmi.Mrdailysynframe_720s synchronic magnetogram series (available at jsoc.stanford.edu/ajax/lookdata.html?ds=hmi.Mrdailysynframe_720s). Apply the paper's timing rule: use the magnetogram for the day of the eruption, but if the eruption occurs before the magnetogram's observation time, use the day before. Record availability per CME (e.g., 09 May 2021 and 22 April 2021 cases) before proceeding, since the downstream PFSS setup — sync2synop.py to shift to the standard Carrington rotation frame, harmonics.py to compute harmonic coefficients, and PFSS.py to build the 0.5°x0.5°x0.1 Rs field grid between 1 Rs and 2.5 Rs — depends on this input. Save the fits file using the HMIYYYYMMDDsync.fits naming convention so subsequent OSPREI scripts can locate it. Note the caveat that a single-day synchronic magnetogram will not capture solar evolution rotating around near the eastern limb, which matters when active regions are in the east.

## Related tools

- **JSOC catalog** (Source catalog queried to confirm HMI synchronic magnetogram availability for the eruption day)
- **SDO/HMI magnetograms (hmi.Mrdailysynframe_720s)** (The specific magnetogram data series whose availability is checked and which feeds the PFSS magnetic background)
- **OSPREI** (Downstream simulation suite whose ForeCAT component requires the PFSS background built from the confirmed magnetogram) — https://github.com/ckay314/OSPREI

## Examples

```
http://jsoc.stanford.edu/ajax/lookdata.html?ds=hmi.Mrdailysynframe_720s
```

## Evaluation signals

- For each CME, the selected magnetogram date follows the timing rule: day of eruption, or the day before when the eruption occurs before the magnetogram time
- The identified series is hmi.Mrdailysynframe_720s from the JSOC catalog, not a different magnetogram product
- Each selected magnetogram maps to a fits file matching the HMIYYYYMMDDsync.fits naming convention expected by sync2synop.py
- Availability is confirmed before any harmonics.py/PFSS.py runs, since those scripts open the fits file by date string
- Eastern-limb active regions near the eruption time are flagged as a known coverage limitation

## Limitations

- A synchronic magnetogram for a single day does not capture solar evolution rotating around near the eastern limb; for both case-study CMEs, active regions were in the east and this effect is unquantified
- The day-before fallback applies only when the eruption occurs before the magnetogram time; otherwise the day-of magnetogram is the recommended input
- Only magnetogram availability is checked here — the PFSS harmonic computation (harmonics.py, ~minutes) and PFSS.py field-grid build (~30–60 min, two ~630 MB pickles) are separate downstream steps run once per day/simulation

## Evidence

- [methods] We recommend using an HMI synchronic magnetogram for the day of the eruption: "We recommend using an HMI synchronic magnetogram for the day of the eruption"
- [discussion] this will not capture any evolution rotating around near the eastern limb: "this will not capture any evolution rotating around near the eastern limb. For both CMEs, we see active regions in the east."
- [readme] We recommend using an HMI synchronic magnetogram for the day of the eruption (or the day before if the eruption occurs before the time of the magnetogram). These are available from the JSOC catalog.: "We recommend using an HMI synchronic magnetogram for the day of the eruption (or the day before if the eruption occurs before the time of the magnetogram). These are available from the JSOC catalog."
- [discussion] SDO/HMI magnetograms are available at http://jsoc.stanford.edu/ajax/lookdata.html?ds=hmi.Mrdailysynframe_720s.: "SDO/HMI magnetograms are available at http://jsoc.stanford.edu/ajax/lookdata.html?ds=hmi.Mrdailysynframe_720s."
