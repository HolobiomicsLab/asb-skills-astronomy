---
name: external-torque-computation-flyby
description: Use when you have a propagated spacecraft-state time series from SISPO's
  Orekit trajectory simulation that presumes perfect attitude control, and you must
  quantify attitude offsets from dust-impact, drag, or gravity-gradient torques.
when_to_use_negative:
- The propagator already models non-ideal attitude dynamics or full external forces
  natively — this add-on exists specifically because the baseline Orekit framework
  presumes a perfect attitude control system
- The goal is optical rendering of dust and gas environments (ComaCreator/Cycles imaging
  pipeline) rather than attitude dynamics — that is a different SISPO plugin use case
- No propagated state series exists (e.g. static imaging or 3-D reconstruction-only
  workflow) — there are no consecutive propagation epochs between which to interleave
  torque computation
tools:
- name: SISPO
  role: 'Host pipeline: provides trajectory simulation and the plugin/input-file mechanism
    into which the perturbation torque module is coupled as a second layer'
  repo: https://github.com/SISPO-developers/sispo
- name: Orekit
  role: Spacecraft-state propagator whose consecutive propagation steps bracket the
    interleaved perturbation-torque computation layer
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pone.0263882
    title: 'SISPO: Space Imaging Simulator for Proximity Operations'
  source_content_hash: sha256:86b43768d3f1b28e2d9cc71cb0798f4d861247b04193c16f54e8623d1b0f9192
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SISPO-developers/sispo
  tools:
  - Orekit
  - SISPO
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# external-torque-computation-flyby

## Summary

An auxiliary add-on layer for SISPO's Orekit-based attitude framework that computes external-force perturbation torques (dust particle impacts, atmospheric drag, gravity gradient) interleaved between spacecraft-state propagation steps, producing per-epoch perturbed attitude records.

## When to use

Use when you have a propagated spacecraft-state time series from SISPO's Orekit trajectory simulation that presumes perfect attitude control, and you must quantify attitude offsets from dust-impact, drag, or gravity-gradient torques.

## When NOT to use

- The propagator already models non-ideal attitude dynamics or full external forces natively — this add-on exists specifically because the baseline Orekit framework presumes a perfect attitude control system
- The goal is optical rendering of dust and gas environments (ComaCreator/Cycles imaging pipeline) rather than attitude dynamics — that is a different SISPO plugin use case
- No propagated state series exists (e.g. static imaging or 3-D reconstruction-only workflow) — there are no consecutive propagation epochs between which to interleave torque computation

## Inputs

- Spacecraft state time series from SISPO's Orekit trajectory propagation (per-epoch positions/attitudes)
- SISPO simulation input file (JSON-style config with 'options' and 'plugins' keys) enabling the perturbation add-on
- Spacecraft/environment parameters for torque computation (dust flux, atmospheric density model, gravity-gradient terms)

## Outputs

- attitude_perturbations.csv: one record per propagated state epoch with columns epoch, dust-impact torque, drag torque, gravity-gradient torque, and resulting attitude offset

## How to apply

Load the spacecraft state time series produced by SISPO's Orekit trajectory propagation (SISPO supports simulation of trajectories per the abstract). Implement a standalone add-on module that computes, for each propagated epoch, the perturbation torques on spacecraft attitude from dust particle impacts, atmospheric drag, and the gravity gradient. Couple the module as an interleaved second layer between consecutive Orekit propagation steps, leaving the baseline perfect-attitude-control propagator unmodified. Run the extended pipeline over the full state series and write the perturbed attitude records to attitude_perturbations.csv. This add-on pattern mirrors SISPO's existing plugin mechanism (e.g. ComaCreator is activated via '--with-plugin' and a plugin path in the input file), so the torque module can be invoked the same way.

## Related tools

- **SISPO** (Host pipeline: provides trajectory simulation and the plugin/input-file mechanism into which the perturbation torque module is coupled as a second layer) — https://github.com/SISPO-developers/sispo
- **Orekit** (Spacecraft-state propagator whose consecutive propagation steps bracket the interleaved perturbation-torque computation layer)

## Examples

```
pip install sispo && sispo --with-sim --with-plugin /app/attitude_perturbations/interface.py
```

## Evaluation signals

- The extended pipeline script executes without errors over the full state series
- attitude_perturbations.csv contains a finite perturbation record for every propagated state epoch (no missing or NaN epochs)
- Each record covers all three external-force sources: dust-impact torque, drag torque, and gravity-gradient torque
- The baseline perfect-attitude-control propagator output is unmodified — the add-on is interleaved between steps rather than altering the propagator itself

## Limitations

- The external-force perturbation package is planned future work, not part of the current released SISPO tool — it must be implemented as a standalone add-on by the practitioner
- SISPO is open access but currently requires basic programming skills to be used, so building and coupling the add-on assumes programming proficiency
- SISPO's plugin mechanism is currently exercised only by ComaCreator ('currently no other plugins exits'), so the torque add-on has no existing reference plugin to copy

## Evidence

- [other] add-on second layer: [evidence span withheld — anchor status: not_found]
- [other] interleaving procedure: [evidence span withheld — anchor status: not_found]
- [abstract] trajectory simulation support: "SISPO allows the simulation of trajectories, light parameters and camera's intrinsic parameters"
- [methods] CLI installation path: "If sispo is installed as a python package via pip, sispo provides a CLI."
- [discussion] accessibility caveat: "The tool is open access and currently requires basic programming skills to be used [33]."
- [readme] plugin activation pattern: ""options": ["--with-render","--with-sim", "--with-plugin"],
    "plugins": ["/app/ComaCreator/interface.py"],"
