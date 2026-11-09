---
name: flux-rope-internal-property-evolution
description: Use when a ForeCAT terminal CME state (final latitude, longitude, tilt,
  coronal speed) must be propagated interplanetary space and the analysis needs internal
  flux rope properties (B, temperature) and expansion/deformation profiles, not just
  arrival time.
when_to_use_negative:
- The goal is coronal deflection/rotation from the solar surface — ForeCAT handles
  that regime; ANTEATR starts from the far corona (FCrmax) and needs an already-evolved
  terminal state.
- The CME is treated as a hydrodynamic pressure pulse with no internal magnetic structure
  — the internal thermal and magnetic evolution machinery would be moot.
- Only a single foreCAT-FIDO in situ profile at 1 AU is needed without propagation/expansion
  modeling — run FIDO directly rather than re-propagating.
tools:
- name: OSPREI
  role: Wrapper invoked with an input text file; with models=ANT it runs ANTEATR-PARADE
    alone from the far-corona state and generates standardized output files.
  repo: https://github.com/ckay314/OSPREI
- name: ANTEATR (PARADE.py)
  role: Performs the interplanetary propagation with drag (SWCd, SWv, SWn, SWB) plus
    expansion and deformation from internal thermal and magnetic forces, fully contained
    in PARADE.py.
  repo: https://github.com/ckay314/OSPREI
- name: ForeCAT
  role: Upstream model whose terminal CME state (final latitude, longitude, tilt,
    coronal speed) is the initial condition for ANTEATR-PARADE.
  repo: https://github.com/ckay314/OSPREI
- name: processOSPREI.py
  role: Processes the standardized ANTEATR output files into a results object and
    generates histograms of the profile quantities.
  repo: https://github.com/ckay314/OSPREI
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1029/2021sw002914
    title: 'OSPREI: A Coupled Approach to Modeling CME‐Driven Space Weather With Automatically
      Generated, User‐Friendly Outputs'
  source_content_hash: sha256:f5f82316a10d22fc8e38f9dcc9737041d6ff69f419231fe6fc3173da6d301d83
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ckay314/OSPREI
  tools:
  - ANTEATR
  - ANTEATR (PARADE.py)
  - ForeCAT
  - OSPREI
  - processOSPREI.py
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# flux-rope-internal-property-evolution

## Summary

Runs the ANTEATR-PARADE component of OSPREI (fully contained in PARADE.py) to propagate a far-corona flux rope CME to the satellite distance while computing the evolution of its internal thermal and magnetic properties via conservation laws. This supplies arrival time plus expansion/deformation physics that simple drag-only arrival models omit.

## When to use

Use when a ForeCAT terminal CME state (final latitude, longitude, tilt, coronal speed) must be propagated interplanetary space and the analysis needs internal flux rope properties (B, temperature) and expansion/deformation profiles, not just arrival time.

## When NOT to use

- The goal is coronal deflection/rotation from the solar surface — ForeCAT handles that regime; ANTEATR starts from the far corona (FCrmax) and needs an already-evolved terminal state.
- The CME is treated as a hydrodynamic pressure pulse with no internal magnetic structure — the internal thermal and magnetic evolution machinery would be moot.
- Only a single foreCAT-FIDO in situ profile at 1 AU is needed without propagation/expansion modeling — run FIDO directly rather than re-propagating.

## Inputs

- ForeCAT terminal CME state (final latitude, longitude, tilt, coronal speed) at FCrmax (~10 Rs)
- OSPREI input text file with date, time, suffix, and models=ANT
- Solar wind drag settings: SWCd, SWv (km/s), SWn (cm^-3), SWB (nT)
- Flux rope parameters: FRpol, FRB/FRT, Gamma, IVDf, FR density/temperature inputs
- Satellite position (SatLat, SatLon, SatR; default 0°, 0°, 213 Rs)

## Outputs

- CME arrival time at the satellite distance
- Profiles with distance of propagation speed and expansion speed
- Profiles of angular widths and aspect ratios (size and shape evolution)
- Profiles of internal magnetic field strength and temperature
- Standardized output files consumable by processOSPREI.py (results object, histograms)

## How to apply

Prepare an OSPREI input text file setting date (YYYYMMDD), time (HH:MM), suffix, and models=ANT so ANTEATR alone runs from the far-corona state with satellite defaults SatLat=0°, SatLon=0°, SatR=213 Rs. Load the ForeCAT terminal CME state into PARADE.py as the interplanetary initial condition. Configure drag toward the background solar wind (SWCd=1, SWv=400 km/s, SWn=5 cm^-3, SWB=6.9 nT — average OMNI values at 1 AU) and flux rope settings FRpol, FRB/FRT scaled from the CME speed, Gamma=1.33, and IVDf=0.5. Run the simulation and record the evolution with distance of propagation and expansion speeds, angular widths and aspect ratios, and internal magnetic field and temperature through to the arrival time. Process standardized output files with processOSPREI.py into a results object and generate histograms of the profile quantities.

## Related tools

- **OSPREI** (Wrapper invoked with an input text file; with models=ANT it runs ANTEATR-PARADE alone from the far-corona state and generates standardized output files.) — https://github.com/ckay314/OSPREI
- **ANTEATR (PARADE.py)** (Performs the interplanetary propagation with drag (SWCd, SWv, SWn, SWB) plus expansion and deformation from internal thermal and magnetic forces, fully contained in PARADE.py.) — https://github.com/ckay314/OSPREI
- **ForeCAT** (Upstream model whose terminal CME state (final latitude, longitude, tilt, coronal speed) is the initial condition for ANTEATR-PARADE.) — https://github.com/ckay314/OSPREI
- **processOSPREI.py** (Processes the standardized ANTEATR output files into a results object and generates histograms of the profile quantities.) — https://github.com/ckay314/OSPREI

## Examples

```
python OSPREI.py OSPtestcase.txt  # input file sets date, time, suffix, models=ANT; then: python processOSPREI.py OSPtestcase.txt
```

## Evaluation signals

- For the 09 May 2021 CME 1 ensemble, mean arrival time ≈ 12 May 2021 18:28 UT (observed 22:00 UT); for CME 2 (22 April 2021), mean arrival ≈ 25 April 2021 01:47 UT — check emitted CSV values fall in reported ranges (value_in_range).
- Mean magnetic obstacle durations of 36 hr (CME 1) and 27 hr (CME 2) fall within the reported ranges.
- Mean peak densities of 11.1 and 5.8 cm^-3 and central B strengths of 18 nT (CME 1) vs 8 nT (CME 2) each fall within reported ranges.
- Output includes both propagation and expansion speeds with distance, not a single radial speed.
- processOSPREI.py successfully builds a results object from the standardized output files without errors.

## Limitations

- Validated on only two CME cases (09 May 2021 and 22 April 2021) with similar coronal origins; more cases needed to establish the model.
- The background solar wind is one of the most under-developed OSPREI components; effects of different backgrounds on results are untested.
- Parameter space has not been optimized — inputs are not fine-tuned, so best attainable results are unknown.
- PFSS/magnetogram setup upstream does not capture solar evolution rotating around near the eastern limb, which can affect both CMEs' source-region inputs.

## Evidence

- [intro] ANTEATR simulates interplanetary propagation including expansion and deformation and determines internal property evolution via conservation laws: "simulates the propagation, including expansion and deformation, of a CME in interplanetary space and determines the evolution of internal properties via conservation laws"
- [readme] README on ANTEATR-PARADE expansion/deformation physics: "It also includes CME expansion and deformation from the internal thermal and magnetic forces, which was added when ANTEATR was updated to ANTEATR-PARADE."
- [readme] README on ANTEATR outputs and containment: "as well as the arrival time and internal thermal and magnetic properties of the CME. This component is fully contained in `PARADE.py`."
- [methods] README on default solar wind values: "All solar wind properties have defaults corresponding to the average OMNI values at 1 AU"
