---
name: initial-condition-parameter-specification
description: Use when running SimAb in single-simulation mode and you must map a target
  mature planet (mass, final orbital distance) onto a full set of initial formation
  parameters before invoking Srun.py.
when_to_use_negative:
- You want randomized exploration of parameter space for a fixed mature planet — use
  the Mrun.py multi-run mode, which randomly assigns initial orbital distance, initial
  core mass, initial planetesimal ratio, and initial dust grain fraction instead.
- You only need to post-process or plot existing run outputs — use the Jupyter plotting
  codes rather than re-specifying initial conditions.
- The target is not a gas giant forming by disk accretion; SimAb is focused specifically
  on atmosphere accretion of gas giant planets.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: Srun.py
  role: Entry point for single-simulation mode; consumes the specified initial conditions
    and mature-planet parameters and writes run(n)/test(m).txt outputs
  repo: github.com/nkhorshid/simab
- name: SimAb
  role: The planet formation simulation whose output folder contains the paper's reference
    parameter values to guide initial-condition choices
  repo: github.com/nkhorshid/simab
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202141455
    title: 'SimAb: A simple, fast, and flexible model to assess the effects of planet
      formation on the atmospheric composition of gas giants'
  source_content_hash: sha256:3b36258bab85ea0a878a4ff7229b1416f5a223befb1d58d939d51f791beaff95
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SimAb
  - Srun.py
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# initial-condition-parameter-specification

## Summary

Specifies the four initial conditions (initial core mass, initial orbital distance, planetesimal ratio, dust grain fraction) plus the mature planet mass and final orbital distance needed to run a single SimAb simulation of gas-giant atmosphere accretion.

## When to use

Use when running SimAb in single-simulation mode and you must map a target mature planet (mass, final orbital distance) onto a full set of initial formation parameters before invoking Srun.py.

## When NOT to use

- You want randomized exploration of parameter space for a fixed mature planet — use the Mrun.py multi-run mode, which randomly assigns initial orbital distance, initial core mass, initial planetesimal ratio, and initial dust grain fraction instead.
- You only need to post-process or plot existing run outputs — use the Jupyter plotting codes rather than re-specifying initial conditions.
- The target is not a gas giant forming by disk accretion; SimAb is focused specifically on atmosphere accretion of gas giant planets.

## Inputs

- Initial core mass of the protoplanet
- Initial orbital distance of the forming planet
- Initial planetesimal ratio
- Initial dust grain fraction
- Mature planet mass
- Final orbital distance of the mature planet
- Reference parameter values from the repository's output folder

## Outputs

- Fully specified parameter set for a single SimAb run (Srun.py invocation)
- run(n) folder containing test(m).txt per-run output file
- run_sum(n).txt final composition summary recorded as [(x/H)_planet]/[(X/H)_sun]

## How to apply

First retrieve the parameter values used in the SimAb paper from the output folder of the nkhorshid/SimAb repository as a reference point. Then assemble the four initial conditions — the core mass, the initial orbital distance, the planetesimal ratio, and the dust grain fraction — together with the mature planet mass and orbital distance; note that the model shows the initial core mass does not influence the final atmospheric composition, while the initial orbital distance controls C/O via differing gas-phase and solid-phase C/O at different distances, and planetesimals drive supersolar metallicity. Pass these values to Srun.py in single-simulation mode, which creates a run(n) folder containing per-run output test(m).txt. Validate by checking that test(m).txt and run_sum(n).txt exist and that the recorded C/O ratio and metallicity ([ (X/H)_planet ]/[ (X/H)_sun ]) are physically plausible and consistent with reference values from the paper's output folder.

## Related tools

- **Srun.py** (Entry point for single-simulation mode; consumes the specified initial conditions and mature-planet parameters and writes run(n)/test(m).txt outputs) — github.com/nkhorshid/simab
- **SimAb** (The planet formation simulation whose output folder contains the paper's reference parameter values to guide initial-condition choices) — github.com/nkhorshid/simab

## Examples

```
python Srun.py  # single-simulation mode: specify core mass, initial orbital distance, planetesimal ratio, dust grain fraction, mature planet mass and final orbital distance
```

## Evaluation signals

- test(m).txt exists inside the created run(n) folder for the executed run
- run_sum(n).txt exists and records composition in the [(X/H)_planet]/[(X/H)_sun] form
- The recorded C/O ratio and metallicity fall within physically plausible ranges
- C/O ratio and metallicity match the reference values deposited with the paper (output folder)
- The parameter set is complete: all four initial conditions plus mature planet mass and final orbital distance were specified

## Limitations

- In this model the initial protoplanet core mass does not influence the final atmospheric composition, so tuning it has no effect on the outcome.
- You must manually create the run(n) folder named after the run number before running; the simulation expects it to exist.
- Single-simulation mode fixes all parameters explicitly; randomized sweeps of the same initial conditions require the separate Mrun.py mode (the paper used 130,000 random runs).

## Evidence

- [readme] The single simulation mode that is run through running Srun.py, by specifying the initial conditions (the core mass, the initial orbital distance, the planetesimal ratio, and the dust grain fraction), and the mature planet mass and orbital distance.: "The single simulation mode that is run through running Srun.py, by specifying the initial conditions (the core mass, the initial orbital distance, the planetesimal ratio, and the dust grain"
- [readme] The final composition is recorded in a file called run_sum(n).txt as [(x/H) of the planet]/[(X/H) of the sun].: "The final composition is recorded in a file called run_sum(n).txt as [(x/H)of the planet]/[(X/H) of the sun]."
- [methods] The values used to write the SimAb paper are available in the output folder.: "The values used to write the SimAb paper are available in the output folder."
- [abstract] We show that the initial protoplanet core mass does not influence the final composition of the planetary atmosphere in the context of our model.: "We show that the initial protoplanet core mass does not influence the final composition of the planetary atmosphere in the context of our model."
