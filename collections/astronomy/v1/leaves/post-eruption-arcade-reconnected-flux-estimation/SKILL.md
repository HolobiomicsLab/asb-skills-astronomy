---
name: post-eruption-arcade-reconnected-flux-estimation
description: Use when a CME case run in OSPREI requires an initial flux-rope magnetic
  field strength (FRB/beta_M) that must be justified from observations rather than
  assumed, and post-eruption arcades are visible in AIA 193/304 Å data for the event.
when_to_use_negative:
- No post-eruption arcade is observable in AIA 193/304 Å for the event (e.g., far-side
  or poorly observed eruptions), so no reconnected flux can be measured
- The goal is a fine-tuned best-fit simulation rather than a forecasting proof-of-concept
  with observationally justifiable inputs
- Only arrival time/speed matters and the CME is treated as a hydrodynamic pressure
  pulse without an internal magnetic structure — FRB/beta_M scaling is then irrelevant
tools:
- name: OSPREI
  role: Coupled Sun-to-Earth modeling suite whose FRB input (flagScales=True, relative
    to background solar wind at start of ANTEATR) is replaced by the derived beta_M;
    run via OSPREI.py with a text input file
  repo: https://github.com/ckay314/OSPREI
- name: ForeCAT
  role: Rerun coronal deflection/rotation stage of the chain with the new FRB scaling
    before handing off to ANTEATR
  repo: https://github.com/ckay314/OSPREI
- name: ANTEATR
  role: Interplanetary propagation (PARADE) whose simulation start defines the background
    solar wind reference level for the flagScales FRB convention
  repo: https://github.com/ckay314/OSPREI
- name: FIDO
  role: Produces the synthetic in situ magnetic field profiles used to compute delta_B
    against the baseline run
  repo: https://github.com/ckay314/OSPREI
- name: processOSPREI.py
  role: Processes standardized OSPREI output files into a results object from which
    arrival time and in situ B are extracted for the derived-vs-assumed comparison
  repo: https://github.com/ckay314/OSPREI
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1029/2021sw002914
    title: 'OSPREI: A Coupled Approach to Modeling CME‐Driven Space Weather With Automatically
      Generated, User‐Friendly Outputs'
  source_content_hash: sha256:c34982c7aab3232b66ce0fa330d10739a278f8a5796214f00be2c58a5657a831
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
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# post-eruption-arcade-reconnected-flux-estimation

## Summary

Estimates an observationally grounded CME flux-rope magnetic field scaling (beta_M) by tracking post-eruption arcade (PEA) area evolution in SDO/AIA 193 and 304 Å imagery and computing reconnected magnetic flux via the Gopalswamy et al. (2017) Flux Rope from Eruption Data method, then feeds the derived value into the OSPREI chain in place of hand-picked FRB scalings.

## When to use

Use when a CME case run in OSPREI requires an initial flux-rope magnetic field strength (FRB/beta_M) that must be justified from observations rather than assumed, and post-eruption arcades are visible in AIA 193/304 Å data for the event.

## When NOT to use

- No post-eruption arcade is observable in AIA 193/304 Å for the event (e.g., far-side or poorly observed eruptions), so no reconnected flux can be measured
- The goal is a fine-tuned best-fit simulation rather than a forecasting proof-of-concept with observationally justifiable inputs
- Only arrival time/speed matters and the CME is treated as a hydrodynamic pressure pulse without an internal magnetic structure — FRB/beta_M scaling is then irrelevant

## Inputs

- SDO/AIA 193 Å post-eruption arcade observations (fits imagery from the SDO/AIA HMI data portal)
- SDO/AIA 304 Å post-eruption arcade observations
- Assumed FRB/beta_M baseline values from the OSPREI input text files (e.g., 4 and 2 for the 09 May and 22 April 2021 case CMEs)
- OSPREI PFSS background setup (HMI synchronic magnetogram, harmonics.py and PFSS.py pickles) for the eruption day

## Outputs

- Reconnected magnetic flux time series per event from the PEA area evolution
- Derived beta_M (initial flux-rope magnetic field scaling relative to background solar wind) per CME
- Processed OSPREI results (arrival time, in situ magnetic field strength) from rerun chains via processOSPREI.py
- betaM_comparison.csv with columns derived_beta_M, assumed_beta_M, delta_arrival_time, delta_B

## How to apply

Download SDO/AIA 193 and 304 Å post-eruption arcade observations for the target event (e.g., the 22 April or 09 May 2021 CMEs) and track the PEA area evolution over time. Compute the reconnected magnetic flux following the Gopalswamy et al. (2017) arcade-flux method, then convert that reconnected flux into a derived initial flux-rope magnetic field scaling (beta_M) expressed relative to the background solar wind at the start of the ANTEATR simulation — matching the OSPREI flagScales=True convention for the FRB parameter. Rerun the full coupled chain (ForeCAT coronal deflection/rotation, ANTEATR interplanetary propagation via PARADE, FIDO in situ synthesis) replacing the assumed FRB scalings (e.g., 4 and 2 in Table 1) with the derived values, then process outputs with processOSPREI.py to extract arrival time and in situ magnetic field strength. Write derived_beta_M, assumed_beta_M, delta_arrival_time, and delta_B to a comparison CSV and assess plausibility of the reconnected fluxes and beta_M against typical arcade-flux magnitudes for the event class by expert review.

## Related tools

- **OSPREI** (Coupled Sun-to-Earth modeling suite whose FRB input (flagScales=True, relative to background solar wind at start of ANTEATR) is replaced by the derived beta_M; run via OSPREI.py with a text input file) — https://github.com/ckay314/OSPREI
- **ForeCAT** (Rerun coronal deflection/rotation stage of the chain with the new FRB scaling before handing off to ANTEATR) — https://github.com/ckay314/OSPREI
- **ANTEATR** (Interplanetary propagation (PARADE) whose simulation start defines the background solar wind reference level for the flagScales FRB convention) — https://github.com/ckay314/OSPREI
- **FIDO** (Produces the synthetic in situ magnetic field profiles used to compute delta_B against the baseline run) — https://github.com/ckay314/OSPREI
- **processOSPREI.py** (Processes standardized OSPREI output files into a results object from which arrival time and in situ B are extracted for the derived-vs-assumed comparison) — https://github.com/ckay314/OSPREI

## Examples

```
python OSPREI.py OSP20210422_derivedBM.txt && python processOSPREI.py OSP20210422_derivedBM.txt
```

## Evaluation signals

- betaM_comparison.csv exists and contains all four required fields: derived_beta_M, assumed_beta_M, delta_arrival_time, delta_B
- Derived reconnected fluxes fall within typical arcade-flux magnitudes for the event class upon expert review
- Derived beta_M values are finite, positive, and plausibly of the same order as the assumed FRB scalings (4 and 2 in Table 1)
- Rerun OSPREI chains complete for both case CMEs (22 April and 09 May 2021) with the derived FRB values, producing standardized output files that processOSPREI.py can parse
- delta_arrival_time and delta_B are computed strictly against the Table 1 baseline runs, not against observations alone

## Limitations

- PEA reconnected-flux-based methods have not previously been tested within OSPREI; determining which such methods work best is explicitly designated as future work
- The hand-picked beta_M values being replaced lack observational justification, so there is no ground truth — the comparison is derived-vs-assumed, not derived-vs-true
- Only two CME cases with relatively similar coronal origins were considered in the OSPREI paper, so generalization of derived beta_M behavior is untested
- Derived beta_M depends on the background solar wind state at the start of the ANTEATR simulation, and the background solar wind is one of the most under-developed OSPREI components

## Evidence

- [readme] FRB flagScales convention: "FRB - The magnetic field strength of the CME flux rope (at the central axis). If flagScales is set to True then it is relative to the background solar wind"
- [readme] flagScales reference level: "at the start of the ANTEATR simulation"
- [readme] processing outputs: "After running OSPREI, the results can be automatically processed using `processOSPREI.py`. This takes the standardized output files, processes them into a results object"
- [intro] case events: [evidence span withheld — anchor status: not_found]
- [discussion] FIDO output: [evidence span withheld — anchor status: not_found]
- [intro] coupling chain: [evidence span withheld — anchor status: not_found]
