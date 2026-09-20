---
name: gas-giant-atmospheric-composition-analysis
description: Use when you need to map a giant planet's formation history (core mass,
  initial orbital distance, planetesimal ratio, dust grain fraction) to its final
  atmospheric C/O ratio and [(X/H)_planet]/[(X/H)_sun] metallicity.
when_to_use_negative:
- You need a self-consistent, physically detailed planet formation model with disk
  evolution and accretion physics — SimAb is explicitly a basic, fast, simplified
  model focused on atmosphere accretion of gas giants.
- 'The target is a terrestrial/icy planet rather than a gas giant: SimAb focuses on
  atmosphere accretion of gas giant planets.'
- You only have the mature planet's observed atmosphere and want inversion of formation
  parameters — this skill runs the forward model; inversion (e.g., bounding initial
  orbital distance from C/O and metallicity) requires the paper's multi-run comparison
  workflow.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: SimAb
  role: Planet formation simulation that computes the final atmospheric composition
    of a gas giant from formation initial conditions
  repo: github.com/nkhorshid/simab
- name: Srun.py
  role: Entry point for single-simulation mode with user-specified initial conditions
    and mature planet mass/orbital distance
  repo: github.com/nkhorshid/simab
- name: Mrun.py
  role: 'Entry point for multi-run mode: user specifies mature planet mass, final
    orbital distance, and number of random runs; initial conditions are randomly assigned'
  repo: github.com/nkhorshid/simab
- name: Jupyter
  role: Plotting and inspection of simulation results
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202141455
    title: 'SimAb: A simple, fast, and flexible model to assess the effects of planet
      formation on the atmospheric composition of gas giants'
  source_content_hash: sha256:24554e7e3e2e9f8969573c7e1bb08998fd527d6379151791c621531c0f6ec607
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - Mrun.py
  - SimAb
  - Srun.py
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# gas-giant-atmospheric-composition-analysis

## Summary

Runs SimAb's single-simulation mode (Srun.py) to predict the final atmospheric composition (C/O ratio and metallicity) of a gas giant given specified formation initial conditions, and reads the resulting run_sum(n).txt composition summary.

## When to use

Use when you need to map a giant planet's formation history (core mass, initial orbital distance, planetesimal ratio, dust grain fraction) to its final atmospheric C/O ratio and [(X/H)_planet]/[(X/H)_sun] metallicity.

## When NOT to use

- You need a self-consistent, physically detailed planet formation model with disk evolution and accretion physics — SimAb is explicitly a basic, fast, simplified model focused on atmosphere accretion of gas giants.
- The target is a terrestrial/icy planet rather than a gas giant: SimAb focuses on atmosphere accretion of gas giant planets.
- You only have the mature planet's observed atmosphere and want inversion of formation parameters — this skill runs the forward model; inversion (e.g., bounding initial orbital distance from C/O and metallicity) requires the paper's multi-run comparison workflow.

## Inputs

- Initial conditions: initial core mass, initial orbital distance, planetesimal ratio, dust grain fraction
- Mature planet mass and final orbital distance
- Reference parameter values used in the SimAb paper (available in the repository's output folder)

## Outputs

- Per-run folder run(n) containing per-run output file test(m).txt
- run_sum(n).txt recording final composition as [(x/H) of the planet]/[(X/H) of the sun] (including C/O ratio and metallicity)
- Plots of results generated with the provided Jupyter codes

## How to apply

Retrieve the parameter values used in the SimAb paper from the repository's output folder, then invoke Srun.py in single-simulation mode, specifying the initial conditions (initial core mass, initial orbital distance, planetesimal ratio, dust grain fraction) together with the mature planet mass and final orbital distance. SimAb creates a per-run folder run(n) and writes per-run output test(m).txt inside it; the final composition summary is recorded in run_sum(n).txt as the ratio of the planet's elemental abundance to the solar abundance, [(x/H) of the planet]/[(X/H) of the sun]. Interpret results with the paper's findings in mind: the initial core mass does not influence the final composition, initial orbital distance drives the C/O ratio (gas-phase vs solid-phase C/O differ with distance), and planetesimals are the main source of supersolar metallicity. For parameter-space exploration instead of a single history, use Mrun.py, which randomly assigns initial conditions and supports 130,000 random runs as used for the paper. Plot and inspect outputs with the provided Jupyter codes.

## Related tools

- **SimAb** (Planet formation simulation that computes the final atmospheric composition of a gas giant from formation initial conditions) — github.com/nkhorshid/simab
- **Srun.py** (Entry point for single-simulation mode with user-specified initial conditions and mature planet mass/orbital distance) — github.com/nkhorshid/simab
- **Mrun.py** (Entry point for multi-run mode: user specifies mature planet mass, final orbital distance, and number of random runs; initial conditions are randomly assigned) — github.com/nkhorshid/simab
- **Jupyter** (Plotting and inspection of simulation results)

## Examples

```
python Srun.py  # single-simulation mode: specify initial core mass, initial orbital distance, planetesimal ratio, dust grain fraction, and mature planet mass and orbital distance
```

## Evaluation signals

- Confirm that the per-run folder run(n) was created and both test(m).txt and run_sum(n).txt exist for the run.
- Check that run_sum(n).txt records the composition in the expected normalized form [(x/H) of the planet]/[(X/H) of the sun].
- Verify the recorded C/O ratio and metallicity fall within physically plausible ranges.
- Compare C/O and metallicity against the reference values deposited with the paper (in the output folder of the repository).
- Sanity-check against paper findings: a dust-grain-dominated planet should have C/O close to the solar value; supersolar metallicity should require substantial planetesimal accretion.

## Limitations

- The initial protoplanet core mass does not influence the final atmospheric composition in this model, so it cannot be constrained from the output.
- SimAb is a basic, simplified model; results depend on its assumptions, detailed in the SimAb paper (arXiv:2111.00279).
- For multi-run mode, the user must manually create the run(n) folder before running; outputs are only written inside it.
- The paper's authors used 130,000 random runs; far fewer random runs will under-sample the parameter space.

## Evidence

- [intro] The single simulation mode that is run through running Srun.py, by specifying the initial conditions: "The single simulation mode that is run through running Srun.py, by specifying the initial conditions"
- [readme] The final composition is recorded in a file called run_sum(n).txt as [(x/H)of the planet]/[(X/H) of the sun].: "The final composition is recorded in a file called run_sum(n).txt as [(x/H)of the planet]/[(X/H) of the sun]."
- [readme] The values used to write the SimAb paper are available in the output folder.: "The values used to write the SimAb paper are available in the output folder."
- [abstract] We show that the initial protoplanet core mass does not influence the final composition of the planetary atmosphere: "We show that the initial protoplanet core mass does not influence the final composition of the planetary atmosphere"
- [readme] I used 130,000 runs for the paper: "the number of the random runs (I used 130,000 runs for the paper)"
