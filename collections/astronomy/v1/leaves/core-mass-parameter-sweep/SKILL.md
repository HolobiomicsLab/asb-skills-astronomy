---
name: core-mass-parameter-sweep
description: Use when a SimAb-based study needs to isolate the effect of one initial
  condition (e.g., initial core mass) on final atmospheric C/O ratio and metallicity,
  keeping all other initial conditions fixed at paper values.
when_to_use_negative:
- When the goal is to explore effects of orbital distance, planetesimal ratio, or
  dust grain fraction — the paper attributes C/O and metallicity variation to those
  parameters, not core mass, so sweeping core mass will not answer that question.
- When randomized exploration of the full initial-condition space is desired — use
  Mrun.py multi-run mode instead, which randomly assigns initial core mass, orbital
  distance, planetesimal ratio, and dust grain fraction.
edam_operation: http://edamontology.org/operation_0244
tools:
- name: SimAb
  role: Planet formation model executed for each core-mass value, with paper parameters
    taken from the output folder
  repo: github.com/nkhorshid/simab
- name: Srun.py
  role: Single simulation mode entry point; run once per core-mass value with all
    other initial conditions fixed
  repo: github.com/nkhorshid/simab
- name: Jupyter
  role: Optional plotting of the C/O ratio and metallicity versus core mass from the
    sweep results
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1051/0004-6361/202141455
    title: 'SimAb: A simple, fast, and flexible model to assess the effects of planet
      formation on the atmospheric composition of gas giants'
  source_content_hash: sha256:54f0b726e02c03ba5553e22bfa53a34204f4de31394fdaeab1ab87a21d7a665a
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
yes_rate: 0.4444444444444444
n_claims: 9
---

# core-mass-parameter-sweep

## Summary

A controlled one-parameter sweep in the SimAb model: run Srun.py in single-simulation mode across the paper's initial core-mass range while holding orbital distance, planetesimal ratio, and dust grain fraction fixed, to test whether core mass affects final C/O ratio and metallicity.

## When to use

Use when a SimAb-based study needs to isolate the effect of one initial condition (e.g., initial core mass) on final atmospheric C/O ratio and metallicity, keeping all other initial conditions fixed at paper values.

## When NOT to use

- When the goal is to explore effects of orbital distance, planetesimal ratio, or dust grain fraction — the paper attributes C/O and metallicity variation to those parameters, not core mass, so sweeping core mass will not answer that question.
- When randomized exploration of the full initial-condition space is desired — use Mrun.py multi-run mode instead, which randomly assigns initial core mass, orbital distance, planetesimal ratio, and dust grain fraction.

## Inputs

- SimAb repository parameters used for the paper (output folder values)
- Initial conditions: core mass (varied), initial orbital distance, planetesimal ratio, dust grain fraction (fixed at paper values)
- Final planet mass and orbital distance for the mature planet

## Outputs

- run_sum(n).txt files recording final composition as [(x/H) of the planet]/[(X/H) of the sun] per run
- core_mass_sweep.csv keyed by initial core mass with C/O ratio and metallicity (relative to solar) columns

## How to apply

Load the SimAb model from github.com/nkhorshid/simab together with the parameter values used to write the SimAb paper, which are available in the repository's output folder. Fix the initial orbital distance, planetesimal ratio, and dust grain fraction at the chosen paper values, then loop over the initial core mass across the paper's tested range, launching one Srun.py single-mode run per core-mass value. Each run writes its final composition to run_sum(n).txt as [(x/H) of the planet]/[(X/H) of the sun]; from these compute the C/O ratio and metallicity relative to solar. Assemble the per-core-mass C/O and metallicity values into core_mass_sweep.csv. The rationale is that SimAb's own paper claims core mass does not influence final atmospheric composition, so a fixed-other-conditions sweep is the direct falsification test of that claim.

## Related tools

- **SimAb** (Planet formation model executed for each core-mass value, with paper parameters taken from the output folder) — github.com/nkhorshid/simab
- **Srun.py** (Single simulation mode entry point; run once per core-mass value with all other initial conditions fixed) — github.com/nkhorshid/simab
- **Jupyter** (Optional plotting of the C/O ratio and metallicity versus core mass from the sweep results)

## Examples

```
for m0 in [10, 20, 50, 100]: run 'python Srun.py' (single mode, core_mass=m0; orbital distance, planetesimal ratio, dust grain fraction fixed); read run_sum(n).txt
```

## Evaluation signals

- C/O ratio column of core_mass_sweep.csv is constant within numerical tolerance across the full core-mass range.
- Metallicity (relative to solar) column of core_mass_sweep.csv is constant within numerical tolerance across the full core-mass range.
- Each run produced a run_sum(n).txt with composition values expressed as [(x/H) of the planet]/[(X/H) of the sun].
- The constant C/O and metallicity values match the paper's reported result that initial core mass does not influence final atmospheric composition.

## Limitations

- The result holds only 'in the context of our model' (SimAb's simplifying assumptions); it is not a general physical statement.
- Only one initial condition is varied per sweep; interactions with orbital distance, planetesimal ratio, and dust grain fraction require separate sweeps or Mrun.py randomized runs.
- README does not document command-line flags for Srun.py, so the core-mass variation mechanism must be inferred from the repository code.

## Evidence

- [abstract] We show that the initial protoplanet core mass does not influence the final composition of the planetary atmosphere in the context of our model.: "We show that the initial protoplanet core mass does not influence the final composition of the planetary atmosphere in the context of our model."
- [readme] The single simulation mode that is run through running Srun.py, by specifying the initial conditions (the core mass, the initial orbital distance, the planetesimal ratio, and the dust grain fraction), and the mature planet mass and orbital distance.: "The single simulation mode that is run through running Srun.py, by specifying the initial conditions (the core mass, the initial orbital distance, the planetesimal"
- [readme] The final composition is recorded in a file called run_sum(n).txt as [(x/H)of the planet]/[(X/H) of the sun].: "The final composition is recorded in a file called run_sum(n).txt as [(x/H)of the planet]/[(X/H) of the sun]."
- [methods] The values used to write the SimAb paper are available in the output folder.: "The values used to write the SimAb paper are available in the output folder."
