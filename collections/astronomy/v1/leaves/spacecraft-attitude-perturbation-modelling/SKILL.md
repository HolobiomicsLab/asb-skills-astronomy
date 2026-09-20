---
name: spacecraft-attitude-perturbation-modelling
description: Use when SISPO/Orekit trajectory propagations must be augmented with
  external-force attitude perturbations (dust impacts, atmospheric drag, gravity gradient)
  that the baseline perfect-attitude-control model ignores.
when_to_use_negative:
- The baseline propagator's perfect-attitude-control assumption is acceptable and
  no external-force perturbation analysis is requested — the add-on is an optional
  second layer, not a replacement
- The task concerns SISPO's imaging/rendering pipeline (Cycles rendering, aberration
  simulation) rather than attitude dynamics
- Only a single epoch is of interest — the method is designed to run over the full
  propagated state series, interleaved between consecutive Orekit steps
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_0621
tools:
- name: SISPO
  role: Provides the full simulation pipeline and Orekit-based trajectory/state propagation
    whose spacecraft states feed the perturbation module
  repo: https://github.com/SISPO-developers/sispo
- name: Orekit
  role: Spacecraft-state propagator between whose consecutive steps the perturbation
    layer is interleaved
- name: sispo_docker
  role: Docker image recommended for deploying SISPO with a ready environment for
    extended pipelines/plugins
  repo: https://github.com/SISPO-developers/sispo_docker
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pone.0263882
    title: 'SISPO: Space Imaging Simulator for Proximity Operations'
  source_content_hash: sha256:321c498da22c2fb0d309098a4bfa07331ed2e0fd4841d14191f71359eb972f55
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SISPO-developers/sispo
  tools:
  - Orekit
  - SISPO
  - sispo_docker
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# spacecraft-attitude-perturbation-modelling

## Summary

Extends SISPO's Orekit-based attitude framework — which presumes perfect attitude control — by interleaving a second-layer add-on module that computes attitude perturbation torques from dust particle impacts, atmospheric drag, and the gravity gradient at every propagated epoch.

## When to use

Use when SISPO/Orekit trajectory propagations must be augmented with external-force attitude perturbations (dust impacts, atmospheric drag, gravity gradient) that the baseline perfect-attitude-control model ignores.

## When NOT to use

- The baseline propagator's perfect-attitude-control assumption is acceptable and no external-force perturbation analysis is requested — the add-on is an optional second layer, not a replacement
- The task concerns SISPO's imaging/rendering pipeline (Cycles rendering, aberration simulation) rather than attitude dynamics
- Only a single epoch is of interest — the method is designed to run over the full propagated state series, interleaved between consecutive Orekit steps

## Inputs

- Spacecraft state time series from SISPO's Orekit trajectory propagation (epoch-tagged states)
- Spacecraft/environment model parameters needed for torque evaluation (e.g. shape/surface for dust impacts and drag, mass distribution for gravity gradient)

## Outputs

- attitude_perturbations.csv with one record per propagated state epoch: epoch, dust-impact torque, drag torque, gravity-gradient torque, resulting attitude offset

## How to apply

Load the spacecraft state time series produced by SISPO's Orekit trajectory propagation. Implement a standalone add-on module that, for each propagated epoch, computes the perturbation torques on spacecraft attitude from the three external-force sources: dust particle impacts, atmospheric drag, and the gravity gradient. Couple the module as an interleaved second layer between consecutive Orekit spacecraft-state propagation steps rather than modifying the baseline perfect-attitude-control propagator. Run the extended pipeline over the full state series and serialise the perturbed attitude records as (epoch, dust-impact torque, drag torque, gravity-gradient torque, resulting attitude offset). Keep the perturbation step optional so the perfect-attitude-control baseline remains available for comparison runs.

## Related tools

- **SISPO** (Provides the full simulation pipeline and Orekit-based trajectory/state propagation whose spacecraft states feed the perturbation module) — https://github.com/SISPO-developers/sispo
- **Orekit** (Spacecraft-state propagator between whose consecutive steps the perturbation layer is interleaved)
- **sispo_docker** (Docker image recommended for deploying SISPO with a ready environment for extended pipelines/plugins) — https://github.com/SISPO-developers/sispo_docker

## Examples

```
sispo --with-render --with-sim --with-plugin
```

## Evaluation signals

- The extended pipeline script executes without errors over the full state series
- attitude_perturbations.csv contains a finite perturbation record for every propagated state epoch
- Every record covers all three external-force sources (dust-impact torque, drag torque, gravity-gradient torque)
- The baseline perfect-attitude-control propagator is left unmodified — identical output when the perturbation layer is disabled
- Resulting attitude offsets are finite and physically bounded for the simulated fly-by/proximity scenario

## Limitations

- The perturbation add-on is a planned extension (future work), described as a second layer on top of the current perfect-attitude-control model, not yet a validated component
- SISPO is open access but currently requires basic programming skills to be used
- External-force effects are modelled only for the three named sources (dust impacts, atmospheric drag, gravity gradient); other torques are out of scope

## Evidence

- [other] External-force effects built as add-ons: [evidence span withheld — anchor status: not_found]
- [other] Interleaved second layer: [evidence span withheld — anchor status: not_found]
- [abstract] SISPO trajectory simulation: "SISPO allows the simulation of trajectories, light parameters and camera’s intrinsic parameters"
- [methods] sispo CLI via pip: "If sispo is installed as a python package via pip, sispo provides a CLI."
- [readme] sispo_docker deployment: [evidence span withheld — anchor status: external_doc]
