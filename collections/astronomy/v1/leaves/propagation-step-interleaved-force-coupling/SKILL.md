---
name: propagation-step-interleaved-force-coupling
description: Use when a spacecraft trajectory time series propagated with SISPO/Orekit
  must be augmented with realistic attitude perturbations from dust impacts, atmospheric
  drag, or gravity-gradient torques that the perfect-attitude-control propagator ignores.
when_to_use_negative:
- The task is image rendering, aberration simulation, or 3-D reconstruction — SISPO's
  core imaging pipeline, which this perturbation layer does not modify.
- The analysis can legitimately assume perfect attitude control (e.g., first-order
  trajectory or rendering studies insensitive to small attitude offsets).
- An inverse attitude-determination problem is posed from real images; this skill
  performs forward perturbation modeling on simulated states only.
edam_operation: http://edamontology.org/operation_3946
tools:
- name: SISPO
  role: Host pipeline whose Orekit-based trajectory propagation produces the spacecraft
    state series the perturbation module is interleaved into
  repo: https://github.com/SISPO-developers/sispo
- name: Orekit
  role: Baseline spacecraft-state propagator; its consecutive propagation steps define
    the interleave points for the perturbation layer
- name: sispo_docker
  role: Recommended containerized environment for running the extended SISPO pipeline
    with add-on plugins
  repo: https://github.com/SISPO-developers/sispo_docker
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pone.0263882
    title: 'SISPO: Space Imaging Simulator for Proximity Operations'
  source_content_hash: sha256:05cad5de42063cb31fdc3d142090e7e766214adbfdfae939fa318b2b9e844efb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SISPO-developers/sispo
  tools:
  - Orekit
  - SISPO
  - sispo_docker
  edam_operation: http://edamontology.org/operation_3946
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Propagation-step interleaved force coupling

## Summary

An auxiliary add-on layer for SISPO's Orekit-based attitude framework that computes external-force perturbation torques (dust particle impacts, atmospheric drag, gravity gradient) for each propagated spacecraft-state epoch and interleaves the results between consecutive Orekit propagation steps, relaxing the baseline perfect-attitude-control assumption.

## When to use

Use when a spacecraft trajectory time series propagated with SISPO/Orekit must be augmented with realistic attitude perturbations from dust impacts, atmospheric drag, or gravity-gradient torques that the perfect-attitude-control propagator ignores.

## When NOT to use

- The task is image rendering, aberration simulation, or 3-D reconstruction — SISPO's core imaging pipeline, which this perturbation layer does not modify.
- The analysis can legitimately assume perfect attitude control (e.g., first-order trajectory or rendering studies insensitive to small attitude offsets).
- An inverse attitude-determination problem is posed from real images; this skill performs forward perturbation modeling on simulated states only.

## Inputs

- Spacecraft state time series from SISPO's Orekit trajectory propagation (one state per epoch)
- Spacecraft physical parameters required for torque computation (surface area, moments of inertia, drag cross-section) for dust-impact, atmospheric-drag, and gravity-gradient models

## Outputs

- attitude_perturbations.csv with columns: epoch, dust-impact torque, drag torque, gravity-gradient torque, resulting attitude offset (one finite record per propagated state epoch)

## How to apply

Load the spacecraft state time series produced by SISPO's Orekit trajectory propagation, then implement a standalone add-on module that evaluates, per epoch, the perturbation torques from the three external-force sources: dust particle impacts, atmospheric drag, and the gravity gradient. Couple the module as an interleaved second layer between consecutive Orekit propagation steps so the baseline perfect-attitude-control propagator remains unmodified — mirroring SISPO's existing plugin pattern (as ComaCreator's interface.py is invoked by Sispo before rendering). For every propagated epoch, write a record containing the epoch, dust-impact torque, drag torque, gravity-gradient torque, and resulting attitude offset to attitude_perturbations.csv. Validation requires the extended pipeline to execute without errors and every propagated state epoch to yield a finite perturbation record covering all three external-force sources.

## Related tools

- **SISPO** (Host pipeline whose Orekit-based trajectory propagation produces the spacecraft state series the perturbation module is interleaved into) — https://github.com/SISPO-developers/sispo
- **Orekit** (Baseline spacecraft-state propagator; its consecutive propagation steps define the interleave points for the perturbation layer)
- **sispo_docker** (Recommended containerized environment for running the extended SISPO pipeline with add-on plugins) — https://github.com/SISPO-developers/sispo_docker

## Examples

```
docker run sispo_docker:latest sispo --with-render --with-sim --with-plugin --plugins /app/attitude_perturbations/interface.py
```

## Evaluation signals

- The extended pipeline script executes end-to-end without errors over the full state series
- attitude_perturbations.csv contains one record for every propagated state epoch (row count equals number of propagation steps)
- Every record is finite (no NaN/Inf) for all three torque columns and the resulting attitude offset
- All three external-force sources (dust impact, drag, gravity gradient) are represented in each record, not just a subset
- The baseline perfect-attitude-control propagator output is bit-identical to runs without the perturbation layer (add-on leaves the core propagator unmodified)

## Limitations

- This perturbation calculation is a planned future-work extension, not a validated core feature of the released SISPO version described in the paper
- Using SISPO and its add-ons requires basic programming skills, which limits accessibility
- The layer only perturbs attitude between propagation steps; trajectory dynamics and the core propagator itself are left unchanged, so coupling fidelity depends on the propagation step size

## Evidence

- [abstract] Orekit state series is the input substrate for trajectory-level simulation in SISPO: "SISPO allows the simulation of trajectories, light parameters and camera's intrinsic parameters"
- [discussion] The perturbation layer is a future-work extension to the released tool: "Some further improvements have been considered and are listed in Section: Discussion and future work."
- [readme] Add-on modules are coupled to Sispo via an interleave/plugin mechanism, the pattern the perturbation layer follows: [evidence span withheld — anchor status: external_doc]
- [readme] Recommended execution environment for running the extended pipeline: [evidence span withheld — anchor status: external_doc]
