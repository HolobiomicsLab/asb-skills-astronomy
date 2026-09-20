---
name: osprei-input-parameter-table-assembly
description: Use when configuring an OSPREI (ForeCAT/ANTEATR/FIDO) simulation of a
  specific CME and you must populate every input parameter with documented values
  and per-parameter source labels from real-time-analogous data rather than fine-tuned
  fits.
when_to_use_negative:
- When the goal is best-fit scientific tuning rather than a real-time-analogous forecast
  — the article explicitly avoids 'fine-tuning input parameters to achieve a best
  fit'
- When the model is not OSPREI/ForeCAT-ANTEATR-FIDO — the parameter names (CMEtilt,
  CMEAW, SWv, FRB) and defaults are OSPREI-specific
- When only arrival time is needed from a simple drag model — for that, coronagraph
  speed plus an arrival-time model suffices without the full 34-parameter table
edam_operation: http://edamontology.org/operation_2422
tools:
- name: OSPREI
  role: Target simulation suite whose 34 input parameters are being populated; defines
    required parameters (date, time, suffix, models, CMElat, CMElon, CMEtilt, CMEvr,
    CMEAW) and default values
  repo: https://github.com/ckay314/OSPREI
- name: OSPREI (ForeCATARPILER fitPIL)
  role: Optional automated extraction of initial CME latitude, longitude, and tilt
    from a PFSS magnetogram to replace purely visual EUV tilt estimates
  repo: https://github.com/ckay314/OSPREI
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1029/2021sw002914
    title: 'OSPREI: A Coupled Approach to Modeling CME‐Driven Space Weather With Automatically
      Generated, User‐Friendly Outputs'
  source_content_hash: sha256:9ee6437282069dc6296d76434df64a924787d9220eacfb7468025a8daaece67a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ckay314/OSPREI
  tools:
  - OSPREI
  - OSPREI (ForeCATARPILER fitPIL)
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# osprei-input-parameter-table-assembly

## Summary

Assembles a complete OSPREI input configuration (34 parameters in an input text/CSV table) for a CME forecasting case, assigning each parameter a provenance label from real-time-analogous observational sources: EUV imagery, DONKI CMEAnalysis entries, GCS reconstructions, OMNI 1 AU solar wind data, or OSPREI defaults.

## When to use

Use when configuring an OSPREI (ForeCAT/ANTEATR/FIDO) simulation of a specific CME and you must populate every input parameter with documented values and per-parameter source labels from real-time-analogous data rather than fine-tuned fits.

## When NOT to use

- When the goal is best-fit scientific tuning rather than a real-time-analogous forecast — the article explicitly avoids 'fine-tuning input parameters to achieve a best fit'
- When the model is not OSPREI/ForeCAT-ANTEATR-FIDO — the parameter names (CMEtilt, CMEAW, SWv, FRB) and defaults are OSPREI-specific
- When only arrival time is needed from a simple drag model — for that, coronagraph speed plus an arrival-time model suffices without the full 34-parameter table

## Inputs

- DONKI CMEAnalysis entries from kauai.ccmc.gsfc.nasa.gov (e.g., 16870 for 09 May 2021; 16772 with IPS 16784 for 22 April 2021)
- OMNI 1 AU solar wind data from omniweb.gsfc.nasa.gov (speed, density, magnetic field strength)
- HMI synchronic magnetogram from the JSOC catalog (hmi.Mrdailysynframe_720s) for the eruption day (or prior day)
- SDO/AIA composite EUV imagery and STEREO SECCHI coronagraph images for visual tilt/geometry estimation
- GCS (Graduated Cylindrical Shell) reconstruction values for CME latitude, longitude, tilt, angular width, and velocity
- OSPREI input-parameter specification (README list of required NUMBER parameters and default values)

## Outputs

- Labeled 34-parameter OSPREI input table (osprei_input_table.csv) with one row per parameter and a per-parameter source-label column (EUV, DONKI, GCS, OMNI, Default)
- OSPREI input text file (e.g., OSPtestcase.txt) with date, time, suffix, models, and CME/solar-wind/flux-rope terms assigned

## How to apply

Retrieve the DONKI CMEAnalysis entry for the target CME (e.g., entry 16870 for the 09 May 2021 CME, and entry 16772 with interplanetary shock 16784 for the 22 April 2021 CME) to anchor CME geometry and kinematics, and pull upstream solar wind speed, density, and magnetic field strength at 1 AU from OMNI. Check JSOC for an HMI synchronic magnetogram on the eruption day (or the day before if the eruption precedes the magnetogram time) to support initial position/tilt via the ForeCATARPILER fitPIL routine or visual EUV estimation. Assign each of the 34 parameters (date, time, suffix, models, CMElat/CMElon/CMEtilt/CMEvr/CMEAW, CMEM, FRB, FRT, FRpol, SWv/SWn/SWB, satellite parameters, etc.) a source label of EUV, DONKI, GCS, OMNI, or Default, falling back to OSPREI's built-in defaults (e.g., SWv 400 km/s, SWn 5 cm^-3, SWB 6.9 nT, average OMNI values at 1 AU) when observational constraints are unavailable. For the reference cases, CME geometry came from EUV/DONKI/GCS (CME 1 tilt 45°, CME 2 tilt 20°, both coronal velocities v_F = 800 km/s), and solar wind values came from OMNI. Write the labeled table (e.g., osprei_input_table.csv) and format the labeled values into the OSPREI input text file consumed by OSPREI.py.

## Related tools

- **OSPREI** (Target simulation suite whose 34 input parameters are being populated; defines required parameters (date, time, suffix, models, CMElat, CMElon, CMEtilt, CMEvr, CMEAW) and default values) — https://github.com/ckay314/OSPREI
- **OSPREI (ForeCATARPILER fitPIL)** (Optional automated extraction of initial CME latitude, longitude, and tilt from a PFSS magnetogram to replace purely visual EUV tilt estimates) — https://github.com/ckay314/OSPREI

## Examples

```
python OSPREI.py OSPtestcase.txt
```

## Evaluation signals

- The labeled CSV contains exactly 34 data rows, one per OSPREI input parameter
- Required fields (date, time, suffix, models, CMElat, CMElon, CMEtilt, CMEvr, CMEAW) and the source-label column are present
- Spot values match reference Table 1: CME 1 tilt 45°, CME 2 tilt 20°, both v_F = 800 km/s
- Every parameter row carries one of the five labels: EUV, DONKI, GCS, OMNI, or Default
- Unset optional parameters are consistent with README defaults (e.g., SWv 400 km/s, SWn 5 cm^-3, SWB 6.9 nT, CMEM 1e15 g, CMEAWp = 0.33 CMEAW)

## Limitations

- Only two CME cases were used to establish the approach; more cases are needed
- Inputs are deliberately not fine-tuned, so best attainable results are unknown without careful parameter-space exploration
- The synchronic magnetogram does not capture solar evolution rotating around near the eastern limb; for both cases active regions in the east could be affected
- Some parameters are hard to measure observationally — e.g., the CME central-axis aspect ratio (CMEdelAx) 'is currently hard if not impossible to get a observationally measure', so it is a guess
- fitPIL / ForeCATARPILER is 'largely untested and should be considered a beta program'

## Evidence

- [intro] forecasting proof-of-concept with real-time-analogous inputs: "We approach these CME as a forecasting proof-of-concept, using information analogous to what would be available in real time rather than fine-tuning input parameters to achieve a best fit"
- [readme] README default solar wind values from OMNI: "All solar wind properties have defaults corresponding to the average OMNI values at 1 AU"
- [methods] magnetogram acquisition rule: "We recommend using an HMI synchronic magnetogram for the day of the eruption"
- [readme] automated PIL/tilt extraction command: "the `fitPIL` command should be passed the date string, x1, x2, y1, y2"
- [readme] required core parameters: "CMElat - The initial latitude of the CME nose in degrees. *[REQUIRED]*"
