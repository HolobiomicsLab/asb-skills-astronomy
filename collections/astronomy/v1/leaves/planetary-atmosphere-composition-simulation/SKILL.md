---
name: planetary-atmosphere-composition-simulation
description: Use when reproducing SimAb experiments on how the initial orbital distance
  of a protoplanet (relative to the H2O and CO ice lines) sets the final atmospheric
  C/O ratio, with fixed core mass, planetesimal ratio, and dust grain fraction.
when_to_use_negative:
- The goal is a randomized parameter-space exploration over mature planet mass and
  final orbital distance — use the multi-run mode (Mrun.py) instead, which randomly
  assigns initial orbital distance, core mass, planetesimal ratio, and dust grain
  fraction
- The question concerns metallicity or planetesimal-driven supersolar enrichment rather
  than the C/O-vs-initial-orbital-distance dependence
- No SimAb installation / run(n) output folder is available and the model cannot be
  executed locally
edam_operation: http://edamontology.org/operation_0244
tools:
- name: SimAb
  role: Planet formation simulation whose single-run physics produces the atmospheric
    composition for each orbital-distance grid point; paper parameter values are in
    its output folder
  repo: github.com/nkhorshid/simab
- name: Srun.py
  role: Single-simulation driver invoked once per initial-orbital-distance grid point,
    given the core mass, initial orbital distance, planetesimal ratio, and dust grain
    fraction
  repo: github.com/nkhorshid/simab
- name: Jupyter
  role: Used to plot the C/O-versus-initial-orbital-distance results
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202141455
    title: 'SimAb: A simple, fast, and flexible model to assess the effects of planet
      formation on the atmospheric composition of gas giants'
  source_content_hash: sha256:28ce5df5b5e0248f214404895f09b922bd696ac2b62b5c6c9cc36487bc34f6b8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - SimAb
  - Srun.py
  edam_operation: http://edamontology.org/operation_0244
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# planetary-atmosphere-composition-simulation

## Summary

A SimAb-based skill for reproducing how a gas giant's final atmospheric C/O ratio depends on its initial orbital distance, by sweeping initial orbital distance across the H2O and CO ice lines while holding core mass, planetesimal ratio, and dust grain fraction fixed.

## When to use

Use when reproducing SimAb experiments on how the initial orbital distance of a protoplanet (relative to the H2O and CO ice lines) sets the final atmospheric C/O ratio, with fixed core mass, planetesimal ratio, and dust grain fraction.

## When NOT to use

- The goal is a randomized parameter-space exploration over mature planet mass and final orbital distance — use the multi-run mode (Mrun.py) instead, which randomly assigns initial orbital distance, core mass, planetesimal ratio, and dust grain fraction
- The question concerns metallicity or planetesimal-driven supersolar enrichment rather than the C/O-vs-initial-orbital-distance dependence
- No SimAb installation / run(n) output folder is available and the model cannot be executed locally

## Inputs

- SimAb repository (github.com/nkhorshid/simab) with Srun.py and the output folder containing the parameter values used for the paper's figures
- Fixed initial conditions: core mass, planetesimal ratio, and dust grain fraction
- Grid of initial orbital distances spanning inside the H2O ice line, between the H2O and CO ice lines, and beyond the CO ice line
- Solar abundance reference for converting [(x/H)_planet]/[(X/H)_sun] entries into absolute C/O ratios

## Outputs

- run(n) folder per simulation containing test(m).txt per-run outputs
- run_sum(n).txt per run recording final composition as [(x/H)_planet]/[(X/H)_sun]
- co_ratio_vs_initial_orbital_distance.csv with one final C/O ratio per orbital-distance grid point

## How to apply

Read the SimAb parameter values used for the paper's figures from the output folder of github.com/nkhorshid/simab, and fix the core mass, planetesimal ratio, and dust grain fraction to those values. Define a grid of initial orbital distances spanning three regimes: inside the H2O ice line, between the H2O and CO ice lines, and beyond the CO ice line, since the gas-phase and solid-phase C/O ratios differ across these regimes. Run one single-mode simulation per grid point via Srun.py, creating a run(n) folder for each run so per-run outputs (test(m).txt) and the final composition in run_sum(n).txt are recorded. Parse each run_sum(n).txt, which stores composition as [(x/H)_planet]/[(X/H)_sun], and convert the recorded carbon and oxygen abundances into a final C/O ratio per orbital distance. Write the resulting C/O-versus-initial-orbital-distance table to co_ratio_vs_initial_orbital_distance.csv, and optionally inspect the trends with the available Jupyter plotting codes.

## Related tools

- **SimAb** (Planet formation simulation whose single-run physics produces the atmospheric composition for each orbital-distance grid point; paper parameter values are in its output folder) — github.com/nkhorshid/simab
- **Srun.py** (Single-simulation driver invoked once per initial-orbital-distance grid point, given the core mass, initial orbital distance, planetesimal ratio, and dust grain fraction) — github.com/nkhorshid/simab
- **Jupyter** (Used to plot the C/O-versus-initial-orbital-distance results)

## Examples

```
python Srun.py  # single run per orbital-distance grid point with fixed core mass, planetesimal ratio, dust grain fraction
```

## Evaluation signals

- co_ratio_vs_initial_orbital_distance.csv contains exactly one C/O value per orbital-distance grid point, covering all three regimes (inside H2O ice line, between H2O and CO ice lines, beyond CO ice line)
- Simulations initiated beyond the CO ice line yield C/O ratios closer to the solar value, matching the paper's reported behavior
- C/O varies with initial orbital distance because gas-phase and solid-phase C/O differ across regimes — the CSV should show regime-dependent offsets, not a constant ratio
- Each run produced a run(n) folder with test(m).txt files and a run_sum(n).txt; no missing outputs for any grid point
- Fixed parameters (core mass, planetesimal ratio, dust grain fraction) are identical across all runs and match the values from the paper's output folder

## Limitations

- SimAb is a deliberately simple, fast, flexible model; results reflect its own formation assumptions rather than a full planet-formation treatment
- Within this model, the initial protoplanet core mass does not influence the final atmospheric composition, so varying it here would not probe composition differences
- The user must manually create the run(n) output folder before running; outputs of a sweep are only complete if this was done for every run

## Evidence

- [abstract] Initial orbital distance affects C/O via gas/solid phase differences: "The initial orbital distance affects the C/O ratio due to the different C/O ratios in the gas phase and the solid phase at different orbital distances."
- [abstract] Beyond-CO-ice-line planets approach solar C/O: "Supersolar metallicity planets that initiate their formation farther than the CO ice line have a C/O ratio closer to the solar value."
- [intro] Single-simulation mode via Srun.py with specified initial conditions: "The single simulation mode that is run through running Srun.py, by specifying the initial conditions (the core mass, the initial orbital distance, the planetesimal ratio, and the dust"
- [intro] Final composition output format: "The final composition is recorded in a file called run_sum(n).txt as [(x/H)of the planet]/[(X/H) of the sun]."
- [readme] Paper parameter values ship in the output folder: "The values used to write the SimAb paper are available in the output folder."
