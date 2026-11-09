---
name: flux-rope-magnetic-field-scaling-parameterization
description: Use when a CME simulation input requires an initial flux-rope magnetic
  field strength (FRB/β_M) and only a hand-assumed value exists, but SDO/AIA 193 and
  304 Å post-eruption arcade observations are available to derive it.
when_to_use_negative:
- The event lacks observable post-eruption arcade signatures in AIA 193/304 Å, so
  reconnected flux cannot be tracked
- The goal is a fine-tuned best-fit scientific study rather than a real-time-analogous
  forecasting proof-of-concept (the article explicitly uses real-time-analogous, not
  fine-tuned, inputs)
- Only arrival time matters and in situ magnetic field is not needed — a simple arrival-time
  model without flux-rope properties would suffice
edam_operation: http://edamontology.org/operation_2426
tools:
- name: OSPREI
  role: Main coupled Sun-to-Earth CME suite whose FRB input parameter (flagScales
    convention) this skill populates observationally; run via OSPREI.py and processed
    with processOSPREI.py
  repo: https://github.com/ckay314/OSPREI
- name: ForeCAT
  role: Coronal deflection/rotation stage rerun with the derived β_M feeding the CME's
    magnetic properties
- name: ANTEATR
  role: Interplanetary propagation stage (PARADE) whose initial FRB scaling, set relative
    to the background solar wind at simulation start under flagScales, is replaced
    with the derived value
- name: FIDO
  role: Produces the in situ magnetic field and velocity profiles used to compute
    delta_B against the baseline run
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1029/2021sw002914
    title: 'OSPREI: A Coupled Approach to Modeling CME‐Driven Space Weather With Automatically
      Generated, User‐Friendly Outputs'
  source_content_hash: sha256:60fb7344ed66a0dc713a8681c48fcf2242d0655298bbc08a6f445fe140396567
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ckay314/OSPREI
  tools:
  - ANTEATR
  - FIDO
  - ForeCAT
  - OSPREI
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# flux-rope-magnetic-field-scaling-parameterization

## Summary

Replaces hand-picked flux-rope magnetic field scalings (the OSPREI FRB/β_M input, e.g. values 4 and 2 for the 09 May and 22 April 2021 case CMEs) with observationally derived values based on post-eruption-arcade (PEA) reconnected flux following the Gopalswamy et al. (2017) Flux Rope from Eruption Data approach.

## When to use

Use when a CME simulation input requires an initial flux-rope magnetic field strength (FRB/β_M) and only a hand-assumed value exists, but SDO/AIA 193 and 304 Å post-eruption arcade observations are available to derive it.

## When NOT to use

- The event lacks observable post-eruption arcade signatures in AIA 193/304 Å, so reconnected flux cannot be tracked
- The goal is a fine-tuned best-fit scientific study rather than a real-time-analogous forecasting proof-of-concept (the article explicitly uses real-time-analogous, not fine-tuned, inputs)
- Only arrival time matters and in situ magnetic field is not needed — a simple arrival-time model without flux-rope properties would suffice

## Inputs

- SDO/AIA 193 Å and 304 Å post-eruption arcade images for the CME(s) of interest
- SDO/HMI magnetograms (e.g. hmi.Mrdailysynframe_720s or HMI synchronic magnetogram from the JSOC catalog) for the magnetic flux measurements
- OSPREI input text file with the baseline (assumed) FRB scaling, e.g. values 4 and 2 for the 09 May and 22 April 2021 case CMEs
- Baseline OSPREI run outputs (arrival time, in situ B) for Table 1 comparison

## Outputs

- Derived β_M (initial flux-rope magnetic field scaling) per CME, in flagScales convention relative to background solar wind at ANTEATR start
- betaM_comparison.csv with columns derived_beta_M, assumed_beta_M, delta_arrival_time, delta_B
- Processed OSPREI results (arrival time, in situ magnetic field strength) for the derived-scaling runs vs. baseline

## How to apply

Download SDO/AIA 193 and 304 Å post-eruption arcade observations for the event, track the PEA area evolution, and compute reconnected magnetic flux with the Gopalswamy et al. (2017) arcade-flux method. Convert the reconnected flux into a derived β_M (flux-rope magnetic field scaling) expressed, per the OSPREI flagScales convention, relative to the background solar wind at the start of the ANTEATR simulation. Rerun the full coupled OSPREI chain — ForeCAT coronal deflection/rotation, ANTEATR (PARADE) interplanetary propagation, FIDO in situ synthesis — replacing the assumed FRB scalings, then process outputs with processOSPREI.py to extract arrival time and in situ magnetic field strength for comparison against the baseline run. The rationale is that hand-picked β_M values lack observational justification, and PEA reconnected-flux methods have not yet been tested in OSPREI, making a derived-vs-assumed comparison the sanctioned extension.

## Related tools

- **OSPREI** (Main coupled Sun-to-Earth CME suite whose FRB input parameter (flagScales convention) this skill populates observationally; run via OSPREI.py and processed with processOSPREI.py) — https://github.com/ckay314/OSPREI
- **ForeCAT** (Coronal deflection/rotation stage rerun with the derived β_M feeding the CME's magnetic properties)
- **ANTEATR** (Interplanetary propagation stage (PARADE) whose initial FRB scaling, set relative to the background solar wind at simulation start under flagScales, is replaced with the derived value)
- **FIDO** (Produces the in situ magnetic field and velocity profiles used to compute delta_B against the baseline run)

## Examples

```
python OSPREI.py OSP20210509_derivedB.txt && python processOSPREI.py OSP20210509_derivedB.txt
```

## Evaluation signals

- betaM_comparison.csv exists and contains exactly the four required fields: derived_beta_M, assumed_beta_M, delta_arrival_time, delta_B
- Derived reconnected fluxes and β_M values pass expert review against typical arcade-flux magnitudes for these event classes
- Derived FRB is expressed relative to the background solar wind at the start of the ANTEATR simulation (flagScales convention), matching how the assumed values 4 and 2 were defined
- Full coupled chain (ForeCAT → ANTEATR → FIDO) reruns successfully for each CME with the derived scaling replacing the assumed FRB
- Delta arrival time and delta B are computable against the processed baseline (Table 1) runs via processOSPREI.py outputs

## Limitations

- PEA reconnected-flux-based methods have not yet been tested in OSPREI; determining which methods work best is explicitly designated as future work
- The hand-picked β_M values being compared against lack observational justification, so deltas reflect one uncertain baseline against another derived estimate
- Only two case CMEs (09 May and 22 April 2021) with relatively similar coronal origins but different in situ properties were considered; more cases are needed
- The CME aspect ratio (CMEdelAx) is noted as 'hard if not impossible' to measure observationally, adding uncertainty to flux-rope geometry alongside the field scaling

## Evidence

- [readme] FRB definition in README: "FRB - The magnetic field strength of the CME flux rope (at the central axis). If flagScales is set to True then it is relative to the background solar wind at the start of the ANTEATR simulation."
- [intro] Magnetic field prediction as open challenge: "Predicting the magnetic field remains an open challenge, most arrival time predictions only include the hydrodynamic properties, such as density and velocity, and an estimate of the maximum Kp value"
- [intro] Real-time-analogous input philosophy: "We approach these CME as a forecasting proof-of-concept, using information analogous to what would be available in real time rather than fine-tuning input parameters to achieve a best fit"
- [discussion] Coupled chain feeding FIDO: [evidence span withheld — anchor status: not_found]
- [readme] Post-processing entry point: "python processOSPREI.py OSPtestcase.txt"
