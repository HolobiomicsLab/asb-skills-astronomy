---
name: solar-extreme-ultraviolet-image-analysis
description: Use when SDO/AIA 193 or 304 Å post-eruption-arcade observations exist
  for a CME and a flux-rope magnetic field scaling (β_M) must be estimated observationally
  instead of assumed by hand (e.g., replacing assumed FRB scalings such as 4 and 2
  from Table 1).
when_to_use_negative:
- The CME has no observable low-coronal post-eruption arcade (e.g., stealth CMEs or
  very quiet-Sun/streamer-belt origins), so the Gopalswamy et al. (2017) PEA-flux
  method has no signal to measure.
- An observed in situ magnetic field strength from a real spacecraft already constrains
  the flux rope directly, removing the need for a PEA-based proxy.
- The goal is only to identify the CME's initial position/orientation from the polarity
  inversion line for ForeCAT — use the ForeCATARPILER/fitPIL workflow instead of PEA
  flux tracking.
edam_operation: http://edamontology.org/operation_3443
tools:
- name: OSPREI
  role: Main coupled Sun-to-Earth suite into which the derived β_M replaces the assumed
    FRB scaling; run via OSPREI.py and processed via processOSPREI.py
  repo: https://github.com/ckay314/OSPREI
- name: ForeCAT
  role: Recomputes the coronal deflection and rotation of the CME when the derived
    magnetic scaling changes the input configuration
  repo: https://github.com/ckay314/OSPREI
- name: ANTEATR
  role: Interplanetary propagation component (PARADE.py) whose start-of-simulation
    background solar wind defines the flagScales normalization of the derived β_M/FRB
  repo: https://github.com/ckay314/OSPREI
- name: FIDO
  role: Produces the synthetic in situ magnetic field and velocity profiles used to
    compute delta_B against the baseline run
  repo: https://github.com/ckay314/OSPREI
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1029/2021sw002914
    title: 'OSPREI: A Coupled Approach to Modeling CME‐Driven Space Weather With Automatically
      Generated, User‐Friendly Outputs'
  source_content_hash: sha256:a7951c472f4ae8a8f4a60473b1409505afc0138a507d50f6f7e1071ecccea743
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ckay314/OSPREI
  tools:
  - ANTEATR
  - FIDO
  - ForeCAT
  - OSPREI
  edam_operation: http://edamontology.org/operation_3443
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# solar-extreme-ultraviolet-image-analysis

## Summary

Analyzes SDO/AIA extreme-ultraviolet post-eruption-arcade (PEA) imagery (193 and 304 Å) to compute reconnected magnetic flux following the Gopalswamy et al. (2017) arcade-flux method, and converts it into an observationally derived flux-rope magnetic field scaling (β_M / FRB) for use in the OSPREI Sun-to-Earth CME modeling chain.

## When to use

Use when SDO/AIA 193 or 304 Å post-eruption-arcade observations exist for a CME and a flux-rope magnetic field scaling (β_M) must be estimated observationally instead of assumed by hand (e.g., replacing assumed FRB scalings such as 4 and 2 from Table 1).

## When NOT to use

- The CME has no observable low-coronal post-eruption arcade (e.g., stealth CMEs or very quiet-Sun/streamer-belt origins), so the Gopalswamy et al. (2017) PEA-flux method has no signal to measure.
- An observed in situ magnetic field strength from a real spacecraft already constrains the flux rope directly, removing the need for a PEA-based proxy.
- The goal is only to identify the CME's initial position/orientation from the polarity inversion line for ForeCAT — use the ForeCATARPILER/fitPIL workflow instead of PEA flux tracking.

## Inputs

- SDO/AIA 193 Å and 304 Å post-eruption-arcade image sequences (FITS) for the CME of interest
- SDO/HMI magnetograms (hmi.Mrdailysynframe_720s / HMI synchronic magnetogram from the JSOC catalog) for the magnetic-flux calculation
- OSPREI input text files (e.g., OSPtestcase.txt) with the baseline assumed FRB values and flagScales convention
- PFSS background pickles produced by harmonics.py and PFSS.py for the eruption day

## Outputs

- Reconnected magnetic flux estimates derived from PEA area evolution per event
- Derived initial flux-rope magnetic field scaling (derived_beta_M) per CME
- betaM_comparison.csv with columns derived_beta_M, assumed_beta_M, delta_arrival_time, and delta_B
- OSPREI single-run or ensemble outputs processed by processOSPREI.py (arrival time, in situ B profiles)

## How to apply

Download the SDO/AIA 193 and 304 Å post-eruption-arcade observations for the event (e.g., the 22 April and 09 May 2021 CMEs) and track the PEA area evolution in each event, then compute the reconnected magnetic flux following the Gopalswamy et al. (2017) arcade-flux method. Convert the reconnected flux into a derived initial flux-rope magnetic field scaling (β_M), expressed relative to the background solar wind at the start of the ANTEATR simulation per the OSPREI flagScales convention (i.e., 'If flagScales is set to True then it is relative to the background solar wind at the start of the ANTEATR simulation'). Rerun the full coupled OSPREI chain — ForeCAT coronal deflection/rotation, ANTEATR interplanetary propagation (PARADE.py), and FIDO in situ synthesis — replacing the assumed FRB scalings with the derived β_M values, using OSPREI's text-file input convention (date, time, suffix, CMElat, CMElon, CMEtilt, CMEvr, CMEAW, CMEM, FRB). Extract arrival time and in situ magnetic field strength with processOSPREI.py and compute differences against the baseline runs to assess whether the derived β_M reproduces the observations as well as or better than the hand-picked values.

## Related tools

- **OSPREI** (Main coupled Sun-to-Earth suite into which the derived β_M replaces the assumed FRB scaling; run via OSPREI.py and processed via processOSPREI.py) — https://github.com/ckay314/OSPREI
- **ForeCAT** (Recomputes the coronal deflection and rotation of the CME when the derived magnetic scaling changes the input configuration) — https://github.com/ckay314/OSPREI
- **ANTEATR** (Interplanetary propagation component (PARADE.py) whose start-of-simulation background solar wind defines the flagScales normalization of the derived β_M/FRB) — https://github.com/ckay314/OSPREI
- **FIDO** (Produces the synthetic in situ magnetic field and velocity profiles used to compute delta_B against the baseline run) — https://github.com/ckay314/OSPREI

## Examples

```
python OSPREI.py OSPtestcase.txt && python processOSPREI.py OSPtestcase.txt
```

## Evaluation signals

- betaM_comparison.csv exists and contains all four required fields: derived_beta_M, assumed_beta_M, delta_arrival_time, and delta_B
- Derived reconnected fluxes and β_M values are assessed by expert review against typical arcade-flux magnitudes for these event classes
- The full coupled chain (ForeCAT → ANTEATR/PARADE → FIDO) completes for each rerun and delta_arrival_time and delta_B are finite relative to the Table 1 baseline runs
- flagScales is set so the derived FRB is interpreted as relative to the background solar wind at the start of the ANTEATR simulation, matching the convention of the assumed values (4 and 2)
- Derived-vs-assumed comparison yields arrival times and in situ fields consistent with the paper's observation that 'The OSPREI "prediction" shows good agreement with the arrival time and in situ properties.'

## Limitations

- PEA reconnected-flux-based methods have not yet been tested in OSPREI; the paper explicitly designates determining which such methods work best as future work, so no validated best-practice recipe exists yet.
- The hand-picked β_M values (4 and 2) lack observational justification, providing no ground truth — validation rests on expert plausibility review rather than quantitative benchmarking.
- Only two CME cases (22 April and 09 May 2021) were considered; the paper notes 'more cases should be considered' before conclusions generalize.
- The PIL-extraction helper ForeCATARPILER is 'largely untested and should be considered a beta program', so automated source-region identification may need visual checking.

## Evidence

- [readme] FRB - The magnetic field strength of the CME flux rope (at the central axis). If flagScales is set to True then it is relative to the background solar wind at the start of the ANTEATR simulation.: "If flagScales is set to True then it is relative to the background solar wind at the start of the ANTEATR simulation."
- [readme] README processOSPREI: "After running OSPREI, the results can be automatically processed using `processOSPREI.py`."
- [other] Article datasets list: "SDO composite imagery is available from https://sdo.gsfc.nasa.gov/data/aiahmi/."
- [methods] Article methods (magnetogram recommendation): "We recommend using an HMI synchronic magnetogram for the day of the eruption"
- [readme] README ForeCATARPILER caveat: "This routine is largely untested and should be considered a beta program, but it gives a good automatically-generated match to what one would visually identify as a PIL."
