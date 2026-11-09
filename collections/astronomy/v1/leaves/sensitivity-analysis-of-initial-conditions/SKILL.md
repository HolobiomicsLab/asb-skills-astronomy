---
name: sensitivity-analysis-of-initial-conditions
description: Use when you need to test whether a specific SimAb initial condition
  (core mass, initial orbital distance, planetesimal ratio, or dust grain fraction)
  changes the final atmospheric composition (C/O ratio and metallicity relative to
  solar) while other inputs are held fixed.
when_to_use_negative:
- When you want the randomized multi-run exploration of many initial conditions via
  Mrun.py instead of a controlled one-factor sweep
- When the question is how initial orbital distance and accreted planetesimals jointly
  determine metallicity (two factors vary simultaneously, so a fixed-others sweep
  is not the right design)
edam_operation: http://edamontology.org/operation_2426
tools:
- name: SimAb
  role: Planet formation model whose final atmospheric composition (C/O ratio, metallicity)
    is probed in the parameter sweep; paper parameter values are in the repository's
    output folder
  repo: https://github.com/nkhorshid/simab
- name: Srun.py
  role: Single simulation mode used to run one simulation per swept initial-condition
    value, specifying initial conditions and mature planet mass/orbital distance
  repo: https://github.com/nkhorshid/simab
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202141455
    title: 'SimAb: A simple, fast, and flexible model to assess the effects of planet
      formation on the atmospheric composition of gas giants'
  source_content_hash: sha256:a41b49cdb4cdb3799e24b35edb9f258037fd3cb2eafe66e667079159bc95a6ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nkhorshid/simab
  tools:
  - SimAb
  - Srun.py
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# sensitivity-analysis-of-initial-conditions

## Summary

Parameter-sweep skill for the SimAb planet-formation model: hold initial orbital distance, planetesimal ratio, and dust grain fraction fixed and vary one initial condition (e.g., core mass) with single-mode Srun.py runs to determine which initial conditions do or do not affect final atmospheric C/O ratio and metallicity.

## When to use

Use when you need to test whether a specific SimAb initial condition (core mass, initial orbital distance, planetesimal ratio, or dust grain fraction) changes the final atmospheric composition (C/O ratio and metallicity relative to solar) while other inputs are held fixed.

## When NOT to use

- When you want the randomized multi-run exploration of many initial conditions via Mrun.py instead of a controlled one-factor sweep
- When the question is how initial orbital distance and accreted planetesimals jointly determine metallicity (two factors vary simultaneously, so a fixed-others sweep is not the right design)

## Inputs

- SimAb model with the paper's parameter values from the repository's output folder
- Chosen fixed initial conditions: initial orbital distance, planetesimal ratio, dust grain fraction
- Sweep range for the varied initial condition (e.g., the paper's tested core-mass range)
- Mature planet mass and final orbital distance for each single-mode run

## Outputs

- Per-run final composition as [(x/H) of the planet]/[(X/H) of the sun] from run_sum(n).txt
- CSV file (e.g., core_mass_sweep.csv) with C/O ratio and metallicity relative to solar keyed by the swept initial condition

## How to apply

Load the SimAb model together with the parameter values used to write the SimAb paper, taken from the repository's output folder. Fix the initial orbital distance, planetesimal ratio, and dust grain fraction at the chosen paper values, then loop over the target initial condition (e.g., core mass) across the paper's tested range, running one single-mode simulation per value through Srun.py with the initial conditions and the mature planet mass and orbital distance specified. Read each run's final composition as [(x/H) of the planet]/[(X/H) of the sun] from run_sum(n).txt, compute the C/O ratio and metallicity relative to solar for each run, and assemble the results keyed by the swept parameter into a CSV (e.g., core_mass_sweep.csv). The rationale is that SimAb maps initial conditions (core mass, initial orbital distance, planetesimal ratio, dust grain fraction) to the mature planet's composition, so a controlled one-factor sweep isolates that condition's influence.

## Related tools

- **SimAb** (Planet formation model whose final atmospheric composition (C/O ratio, metallicity) is probed in the parameter sweep; paper parameter values are in the repository's output folder) — https://github.com/nkhorshid/simab
- **Srun.py** (Single simulation mode used to run one simulation per swept initial-condition value, specifying initial conditions and mature planet mass/orbital distance) — https://github.com/nkhorshid/simab

## Examples

```
python Srun.py  # single-mode run: fixed orbital distance, planetesimal ratio, dust grain fraction; loop over core mass and collect run_sum(n).txt
```

## Evaluation signals

- C/O ratio and metallicity columns in core_mass_sweep.csv are constant within numerical tolerance across the full core-mass range, reproducing the paper's finding that initial core mass does not influence final composition
- Each run produces a run(n) folder with the final composition written to run_sum(n).txt in [(x/H) of the planet]/[(X/H) of the sun] form
- Values used match the parameters stored in the repository's output folder that were used to write the SimAb paper
- Computed metallicity is expressed relative to solar abundances, consistent with the paper's subsolar/supersolar terminology
- Only the swept parameter varies between runs; orbital distance, planetesimal ratio, and dust grain fraction remain at their fixed paper values

## Limitations

- The core-mass-independence result holds in the context of the SimAb model's assumptions, not necessarily for other formation models
- Influential parameters such as initial orbital distance and planetesimal ratio act jointly (subsolar vs supersolar metallicity depends on both), so a one-factor-at-a-time sweep cannot capture their interaction
- Each swept value requires a separate Srun.py run with its own run(n) output folder; results are not obtained from a single multi-run invocation

## Evidence

- [readme] The single simulation mode that is run through running Srun.py, by specifying the initial conditions (the core mass, the initial orbital distance, the planetesimal ratio, and the dust grain: "The single simulation mode that is run through running Srun.py, by specifying the initial conditions (the core mass, the initial orbital distance, the planetesimal ratio, and the dust grain"
- [readme] Final composition file format: "The final composition is recorded in a file called run_sum(n).txt as [(x/H)of the planet]/[(X/H) of the sun]."
- [abstract] Core-mass independence finding: "We show that the initial protoplanet core mass does not influence the final composition of the planetary atmosphere in the context of our model."
- [methods] Paper parameter values location: "The values used to write the SimAb paper are available in the output folder."
