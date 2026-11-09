---
name: coupled-cme-model-rerun-configuration
description: Use when a new estimate exists for an OSPREI input parameter (e.g., a
  PEA-reconnected-flux-derived β_M) and you need to quantify its effect on CME arrival
  time and in situ B relative to assumed baseline values.
when_to_use_negative:
- You only need arrival time from a simple model without internal magnetic structure
  — OSPREI's coupled chain is heavier than hydrodynamic-only propagation (current
  forecasts often treat the CME as a pressure pulse)
- No PFSS background exists for the eruption date and you cannot obtain an HMI synchronic
  magnetogram — ForeCAT cannot run without the precomputed background pickles
- You are fine-tuning inputs to best-fit observations rather than testing a hypothesis
  — the paper's proof-of-concept explicitly uses real-time-analogous information rather
  than fine-tuned inputs
tools:
- name: OSPREI
  role: Main wrapper (`OSPREI.py`) that runs the coupled ForeCAT→ANTEATR→FIDO chain
    from a text input file and generates systematic output files
  repo: https://github.com/ckay314/OSPREI
- name: ForeCAT
  role: Simulates coronal deflection and rotation using the PFSS background; its final
    latitude/longitude/tilt feed ANTEATR
  repo: https://github.com/ckay314/OSPREI
- name: ANTEATR
  role: Propagates the CME with drag, expansion, and deformation (PARADE) and yields
    arrival time and internal magnetic properties affected by the FRB scaling
  repo: https://github.com/ckay314/OSPREI
- name: FIDO
  role: Produces synthetic in situ magnetic field and velocity profiles (and Kp) at
    the spacecraft, whose B strength reflects the changed FRB/β_M
  repo: https://github.com/ckay314/OSPREI
- name: processOSPREI.py
  role: Post-processing script that loads standardized output files into a results
    object for extracting arrival time and in situ B for the delta comparison
  repo: https://github.com/ckay314/OSPREI
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1029/2021sw002914
    title: 'OSPREI: A Coupled Approach to Modeling CME‐Driven Space Weather With Automatically
      Generated, User‐Friendly Outputs'
  source_content_hash: sha256:3b1c7d0158b802aba77dcbb25b8c57d0229ab7301058a5107001759a3e838477
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ckay314/OSPREI
  tools:
  - ANTEATR
  - FIDO
  - ForeCAT
  - OSPREI
  - processOSPREI.py
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# coupled-cme-model-rerun-configuration

## Summary

Configure and rerun the full coupled OSPREI chain (ForeCAT coronal deflection/rotation, ANTEATR-PARADE interplanetary propagation, FIDO in situ synthesis) with one or more input parameters replaced (e.g., swapping a hand-assumed FRB/β_M flux-rope magnetic scaling for an observationally derived value), then extract and diff arrival times and in situ magnetic fields against baseline runs.

## When to use

Use when a new estimate exists for an OSPREI input parameter (e.g., a PEA-reconnected-flux-derived β_M) and you need to quantify its effect on CME arrival time and in situ B relative to assumed baseline values.

## When NOT to use

- You only need arrival time from a simple model without internal magnetic structure — OSPREI's coupled chain is heavier than hydrodynamic-only propagation (current forecasts often treat the CME as a pressure pulse)
- No PFSS background exists for the eruption date and you cannot obtain an HMI synchronic magnetogram — ForeCAT cannot run without the precomputed background pickles
- You are fine-tuning inputs to best-fit observations rather than testing a hypothesis — the paper's proof-of-concept explicitly uses real-time-analogous information rather than fine-tuned inputs

## Inputs

- OSPREI text input file (key=value parameters) with date, time, suffix, CMElat, CMElon, CMEtilt, CMEvr, CMEAW plus the modified FRB/β_M value
- PFSS harmonic coefficients and pickle files from HMI synchronic magnetogram for the eruption day (HMIYYYYMMDDsync.fits via JSOC)
- Baseline OSPREI run outputs (assumed FRB values, e.g., 4 and 2 for the 09 May and 22 April 2021 CMEs) for delta computation

## Outputs

- Systematic OSPREI output files (ForeCAT final latitude/longitude/tilt, ANTEATR arrival time, speed, size, shape, internal thermal/magnetic properties, FIDO in situ B and velocity profiles, Kp)
- Processed results object and figures from processOSPREI.py
- Comparison table (e.g., betaM_comparison.csv with derived_beta_M, assumed_beta_M, delta_arrival_time, delta_B)

## How to apply

Write a fully specified OSPREI text input file with REQUIRED parameters (date YYYYMMDD, time HH:MM, suffix, CMElat, CMElon, CMElon in Carrington coordinates, CMEtilt, CMEvr, CMEAW) plus the modified parameter — e.g., set FRB to the derived β_M value with flagScales=True so it is expressed relative to the background solar wind at the start of the ANTEATR simulation. Ensure the PFSS background is already built for the eruption date (sync2synop.py → harmonics.py → PFSS.py, which writes two ~630 MB pickles interpolatable by ForeCAT on a 0.5°x0.5°x0.1 Rs grid between 1 and 2.5 Rs). Run `python OSPREI.py <inputfile>.txt` (set nRuns=1 for a single deterministic rerun; an ensemble needs a matching .ens file), then `python processOSPREI.py <inputfile>.txt` to obtain processed results. Use distinct `suffix` values for baseline vs. rerun so OSPREI's date-based output folders avoid overwriting, then extract arrival time and in situ magnetic field strength and compute deltas (e.g., delta_arrival_time, delta_B) relative to the Table 1 baseline runs. Keep unchanged parameters (SWv=400 km/s, SWn=5 cm^-3, SWB=6.9 nT defaults; FCrmax=10 Rs for normal CMEs) fixed so the comparison isolates the modified scaling.

## Related tools

- **OSPREI** (Main wrapper (`OSPREI.py`) that runs the coupled ForeCAT→ANTEATR→FIDO chain from a text input file and generates systematic output files) — https://github.com/ckay314/OSPREI
- **ForeCAT** (Simulates coronal deflection and rotation using the PFSS background; its final latitude/longitude/tilt feed ANTEATR) — https://github.com/ckay314/OSPREI
- **ANTEATR** (Propagates the CME with drag, expansion, and deformation (PARADE) and yields arrival time and internal magnetic properties affected by the FRB scaling) — https://github.com/ckay314/OSPREI
- **FIDO** (Produces synthetic in situ magnetic field and velocity profiles (and Kp) at the spacecraft, whose B strength reflects the changed FRB/β_M) — https://github.com/ckay314/OSPREI
- **processOSPREI.py** (Post-processing script that loads standardized output files into a results object for extracting arrival time and in situ B for the delta comparison) — https://github.com/ckay314/OSPREI

## Examples

```
python OSPREI.py OSP20210422_betaM.txt && python processOSPREI.py OSP20210422_betaM.txt
```

## Evaluation signals

- Both baseline and rerun outputs exist under distinct suffix-tagged, date-based folders (no overwriting), confirming the suffix parameter was set correctly
- The comparison CSV exists and contains the four required fields: derived_beta_M, assumed_beta_M, delta_arrival_time, delta_B
- Derived reconnected fluxes and β_M values pass expert plausibility review against typical arcade-flux magnitudes for these event classes
- Unchanged parameters (SWv, SWn, SWB, FCrmax=10 Rs, CMEr=1.1 Rs defaults) match the baseline input files, isolating the modified scaling as the only difference
- flagScales=True is confirmed so FRB is interpreted as relative to the background solar wind at the start of the ANTEATR simulation, matching the baseline convention

## Limitations

- Full Sun-to-Earth simulations take ~half a minute per case, but the PFSS background setup (harmonics.py, PFSS.py) takes up to an hour and only needs to run once per day/simulation
- ForeCAT ARPILER/PIL-fitting is a largely untested beta program; initial position inputs may need visual verification
- Only two case CMEs (22 April and 09 May 2021) were studied; whether OSPREI's in situ magnetic field reproduction is a persistent limitation is unresolved, and parameter space has not been optimized
- FIDO currently assumes a Parker Spiral orientation for upstream direction and cannot predict actual in situ magnetic field components

## Evidence

- [readme] FRB - The magnetic field strength of the CME flux rope (at the central axis). If flagScales is set to True then it is relative to the background solar wind at the start of the ANTEATR simulation.: "FRB - The magnetic field strength of the CME flux rope (at the central axis). If flagScales is set to True then it is relative to the background solar wind at the start of the ANTEATR simulation."
- [readme] OSPREI runs ForeCAT for coronal CME deflection and rotation, ANTEATR (PARADE) for interplanetary CME propagation, expansion, and deformation, and FIDO for in situ profiles.: "OSPREI runs ForeCAT for coronal CME deflection and rotation, ANTEATR (PARADE) for interplanetary CME propagation, expansion, and deformation, and FIDO for in situ profiles."
- [readme] OSPREI automatically generates file names based on the date and groups them in folders. This allows for different configurations for the same case: "OSPREI automatically generates file names based on the date and groups them in folders. This allows for different configurations for the same case"
- [discussion] Now that they are fully coupled, full Sun-to-Earth CME simulations can be run in about half a minute per case.: "Now that they are fully coupled, full Sun-to-Earth CME simulations can be run in about half a minute per case."
- [intro] We approach these CME as a forecasting proof-of-concept, using information analogous to what would be available in real time rather than fine-tuning input parameters: "We approach these CME as a forecasting proof-of-concept, using information analogous to what would be available in real time rather than fine-tuning input parameters to achieve a best fit"
