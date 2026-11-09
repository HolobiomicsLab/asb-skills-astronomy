---
name: orbital-distance-parameter-sweep
description: Use when you need to reproduce or extend the dependence of a giant planet's
  final atmospheric C/O ratio on its initial formation distance in the SimAb model,
  e.g. testing how positioning formation inside vs. beyond the CO ice line shifts
  C/O toward the solar value.
when_to_use_negative:
- You need randomized sampling of all initial conditions for a fixed mature planet
  mass and final orbital distance — use the Mrun.py multi-run mode instead of a controlled
  orbital-distance sweep.
- You want to study sensitivity to dust grain fraction or planetesimal ratio rather
  than initial orbital distance — those are separate effects in SimAb and require
  their own sweeps with orbital distance held fixed.
edam_operation: http://edamontology.org/operation_2426
tools:
- name: SimAb
  role: Planet formation simulation that computes final atmospheric composition for
    each initial orbital distance in the sweep
  repo: https://github.com/nkhorshid/SimAb
- name: Srun.py
  role: Single-simulation mode used to run one SimAb simulation per orbital-distance
    grid point with fixed initial conditions
  repo: https://github.com/nkhorshid/SimAb
- name: Jupyter
  role: Plotting the C/O-versus-initial-orbital-distance results
  repo: https://github.com/nkhorshid/SimAb
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202141455
    title: 'SimAb: A simple, fast, and flexible model to assess the effects of planet
      formation on the atmospheric composition of gas giants'
  source_content_hash: sha256:db0148a64ac51be179756b403355eb4c888600235bed8809efdd0fbf364c99bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nkhorshid/SimAb
  tools:
  - Jupyter
  - SimAb
  - Srun.py
  edam_operation: http://edamontology.org/operation_2426
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# orbital-distance-parameter-sweep

## Summary

Sweeps SimAb simulations over a grid of initial protoplanet orbital distances spanning inside the H2O ice line, between the H2O and CO ice lines, and beyond the CO ice line, to quantify how initial orbital distance controls the final atmospheric C/O ratio.

## When to use

Use when you need to reproduce or extend the dependence of a giant planet's final atmospheric C/O ratio on its initial formation distance in the SimAb model, e.g. testing how positioning formation inside vs. beyond the CO ice line shifts C/O toward the solar value.

## When NOT to use

- You need randomized sampling of all initial conditions for a fixed mature planet mass and final orbital distance — use the Mrun.py multi-run mode instead of a controlled orbital-distance sweep.
- You want to study sensitivity to dust grain fraction or planetesimal ratio rather than initial orbital distance — those are separate effects in SimAb and require their own sweeps with orbital distance held fixed.

## Inputs

- SimAb parameter values used for the paper's figures (from the output folder of the SimAb repository)
- Grid of initial orbital distances spanning inside the H2O ice line, between the H2O and CO ice lines, and beyond the CO ice line
- Fixed initial core mass, planetesimal ratio, and dust grain fraction

## Outputs

- run_sum(n).txt composition files per simulation, recorded as [(x/H) of the planet]/[(X/H) of the sun]
- co_ratio_vs_initial_orbital_distance.csv with one C/O ratio per orbital-distance grid point

## How to apply

Read the SimAb parameter values the paper used from the output folder and fix the initial core mass, planetesimal ratio, and dust grain fraction, since the paper shows the initial core mass does not influence the final composition. Define a grid of initial orbital distances that deliberately spans the three regimes delimited by the H2O and CO ice lines, because gas-phase and solid-phase C/O differ across those regimes. Run one simulation per grid point through Srun.py (single simulation mode), collecting the run_sum(n).txt outputs that record final composition as [(x/H) of the planet]/[(X/H) of the sun]. Convert the recorded (X/H) ratios into a final C/O ratio per orbital distance, write co_ratio_vs_initial_orbital_distance.csv, and check that the trend matches the paper's finding that planets initiating formation farther than the CO ice line have C/O closer to the solar value.

## Related tools

- **SimAb** (Planet formation simulation that computes final atmospheric composition for each initial orbital distance in the sweep) — https://github.com/nkhorshid/SimAb
- **Srun.py** (Single-simulation mode used to run one SimAb simulation per orbital-distance grid point with fixed initial conditions) — https://github.com/nkhorshid/SimAb
- **Jupyter** (Plotting the C/O-versus-initial-orbital-distance results) — https://github.com/nkhorshid/SimAb

## Examples

```
python Srun.py  # single simulation mode: fix core mass, planetesimal ratio, dust grain fraction; vary initial orbital distance across the H2O/CO ice-line grid
```

## Evaluation signals

- co_ratio_vs_initial_orbital_distance.csv contains exactly one C/O value per orbital-distance grid point.
- Grid points inside the H2O ice line, between the H2O and CO ice lines, and beyond the CO ice line show distinct C/O ratios, reflecting the different gas-phase and solid-phase C/O across regimes.
- Grid points beyond the CO ice line yield C/O ratios closer to the solar value, reproducing the paper's reported behavior.
- Core mass, planetesimal ratio, and dust grain fraction are held fixed across all sweep runs, matching the paper's parameter choices from the output folder.
- Each run directory run(n) contains test(m).txt per-run outputs and a run_sum(n).txt final composition in [(x/H) of the planet]/[(X/H) of the sun] format.

## Limitations

- The sweep relies on SimAb's simple parameterized formation model; only initial orbital distance varies, so results do not capture other formation pathways.
- The CO ice line location must be known or assumed to place grid points in the correct regimes; misplacing the ice lines invalidates the interpretation.
- Only the supersolar-metallicity, formation-beyond-CO-ice-line regime gives C/O closer to solar; conclusions about C/O as a distance constraint apply specifically to supersolar metallicity planets.

## Evidence

- [abstract] Initial orbital distance affects the C/O ratio due to differing gas-phase and solid-phase C/O at different orbital distances: "The initial orbital distance affects the C/O ratio due to the different C/O ratios in the gas phase and the solid phase at different orbital distances."
- [abstract] Supersolar metallicity planets forming beyond the CO ice line have C/O closer to solar: "Supersolar metallicity planets that initiate their formation farther than the CO ice line have a C/O ratio closer to the solar value."
- [abstract] The initial protoplanet core mass does not influence the final atmospheric composition in the model: "We show that the initial protoplanet core mass does not influence the final composition of the planetary atmosphere in the context of our model."
- [readme] Srun.py single simulation mode with specified initial conditions: "The single simulation mode that is run through running Srun.py, by specifying the initial conditions (the core mass, the initial orbital distance, the planetesimal ratio, and the dust grain fraction)"
- [readme] Paper parameter values available in the output folder: "The values used to write the SimAb paper are available in the output folder."
