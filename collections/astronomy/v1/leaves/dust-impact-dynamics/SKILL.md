---
name: dust-impact-dynamics
description: Use when propagating a spacecraft attitude profile around a small body
  (e.g., comet/asteroid fly-by) and you must quantify how dust particle impacts, atmospheric
  drag, or gravity-gradient torques perturb an attitude solution that would otherwise
  be treated as perfectly controlled.
when_to_use_negative:
- The study explicitly presumes a perfect attitude control system and only needs trajectory/imaging
  simulation — adding the perturbation layer would confound the baseline model
- No external-force sources are physically relevant for the scenario (e.g., no atmosphere
  and no dust/coma environment near the target body), so two of the three torque terms
  are undefined
- The task is dust/gas optical rendering for imagery rather than mechanical perturbation
  of attitude — use ComaCreator's rendering path instead of the attitude perturbation
  layer
edam_operation: http://edamontology.org/operation_2426
tools:
- name: SISPO
  role: Produces the spacecraft state time series from trajectory propagation that
    the perturbation layer interleaves
  repo: https://github.com/SISPO-developers/sispo
- name: Orekit
  role: Baseline spacecraft-state and attitude propagator between whose consecutive
    steps the external-force perturbation module is inserted
- name: ComaCreator
  role: Dust and gas environment generator whose dust particle field can feed the
    dust-impact torque term for cometary/active-body scenarios
  repo: https://github.com/SISPO-developers/ComaCreator
- name: sispo_docker
  role: Recommended Docker environment that ships Sispo (and ComaCreator dependencies)
    ready to run the extended pipeline
  repo: https://github.com/SISPO-developers/sispo_docker
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pone.0263882
    title: 'SISPO: Space Imaging Simulator for Proximity Operations'
  source_content_hash: sha256:be9c5e02fb469214b596c4c44509cbbd8f36e7dccc09983b3272dfc4de025ad5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SISPO-developers/sispo
  tools:
  - ComaCreator
  - Orekit
  - SISPO
  - sispo_docker
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# dust-impact-dynamics

## Summary

A second-layer perturbation module that computes dust-impact, atmospheric-drag, and gravity-gradient torques on spacecraft attitude between Orekit spacecraft-state propagation steps, relaxing the perfect-attitude-control assumption in SISPO-based mission simulations.

## When to use

Use when propagating a spacecraft attitude profile around a small body (e.g., comet/asteroid fly-by) and you must quantify how dust particle impacts, atmospheric drag, or gravity-gradient torques perturb an attitude solution that would otherwise be treated as perfectly controlled.

## When NOT to use

- The study explicitly presumes a perfect attitude control system and only needs trajectory/imaging simulation — adding the perturbation layer would confound the baseline model
- No external-force sources are physically relevant for the scenario (e.g., no atmosphere and no dust/coma environment near the target body), so two of the three torque terms are undefined
- The task is dust/gas optical rendering for imagery rather than mechanical perturbation of attitude — use ComaCreator's rendering path instead of the attitude perturbation layer

## Inputs

- Spacecraft state time series from SISPO's Orekit trajectory propagation (epoch-indexed states)
- Optional dust particle environment field (e.g., ComaCreator dust particle output) for dust-impact torque computation
- Spacecraft surface/geometry and mass-properties description for drag and gravity-gradient torque models

## Outputs

- attitude_perturbations.csv: per-epoch records of (epoch, dust-impact torque, drag torque, gravity-gradient torque, resulting attitude offset)

## How to apply

Load the spacecraft state time series produced by SISPO's Orekit trajectory propagation (SISPO allows simulation of trajectories). Implement a standalone add-on module that, for each propagated epoch, computes the perturbation torques on attitude from three sources: dust particle impacts (dust environments can be generated with ComaCreator for cometary/active-body scenarios), atmospheric drag, and the gravity gradient. Couple this module as an interleaved second layer between consecutive Orekit propagation steps rather than modifying the baseline perfect-attitude-control propagator, so the clean trajectory/attitude solution remains available for comparison. Run the extended pipeline over the full state series and write one perturbed-attitude record per epoch (epoch, dust-impact torque, drag torque, gravity-gradient torque, resulting attitude offset) to attitude_perturbations.csv.

## Related tools

- **SISPO** (Produces the spacecraft state time series from trajectory propagation that the perturbation layer interleaves) — https://github.com/SISPO-developers/sispo
- **Orekit** (Baseline spacecraft-state and attitude propagator between whose consecutive steps the external-force perturbation module is inserted)
- **ComaCreator** (Dust and gas environment generator whose dust particle field can feed the dust-impact torque term for cometary/active-body scenarios) — https://github.com/SISPO-developers/ComaCreator
- **sispo_docker** (Recommended Docker environment that ships Sispo (and ComaCreator dependencies) ready to run the extended pipeline) — https://github.com/SISPO-developers/sispo_docker

## Examples

```
python attitude_perturbation_pipeline.py --states orekit_states.csv --with-dust --with-drag --with-gravity --out attitude_perturbations.csv
```

## Evaluation signals

- The extended pipeline script executes without errors over the full propagated state series
- attitude_perturbations.csv contains one finite perturbation record for every propagated state epoch
- Each record covers all three external-force sources (dust-impact torque, drag torque, gravity-gradient torque)
- With the perturbation module disabled, the baseline propagator output is byte-identical to the original perfect-attitude-control trajectory (the baseline was left unmodified)
- Resulting attitude offsets remain finite and physically plausible (torque magnitudes consistent with the input dust/drag/gravity-gradient models)

## Limitations

- The current Orekit-based attitude framework presumes a perfect attitude control system; the external-force perturbation step is a planned future-work add-on, not part of the released SISPO baseline
- Dust environments from ComaCreator are explicitly development-stage and intended for creating realistic-looking pictures, not science-grade dust dynamics, so dust-impact torques derived from them carry high uncertainty
- Using SISPO (and thus the perturbation pipeline) requires basic programming skills, and no changelog is available for the source repository

## Evidence

- [abstract] SISPO supports trajectory simulation feeding the perturbation layer: "SISPO allows the simulation of trajectories, light parameters and camera’s intrinsic parameters"
- [abstract] Dust environments modelable for comet/active-body scenarios: "has supplementary models to produce realistic dust- and gas-environment optical models for comets and active asteroids"
- [readme] ComaCreator supplies the dust/gas environment used by Sispo: [evidence span withheld — anchor status: external_doc]
- [readme] ComaCreator dust output is not science-grade for dynamics: [evidence span withheld — anchor status: external_doc]
- [discussion] Perturbation add-ons sit in the future-work discussion: "Some further improvements have been considered and are listed in Section: Discussion and future work."
- [discussion] Accessibility limitation for implementing the add-on module: "The tool is open access and currently requires basic programming skills to be used"
