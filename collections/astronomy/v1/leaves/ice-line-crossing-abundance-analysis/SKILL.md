---
name: ice-line-crossing-abundance-analysis
description: Use when studying how initial formation location (inside the H2O ice
  line, between H2O and CO ice lines, or beyond the CO ice line) determines the final
  C/O ratio of a gas giant's atmosphere in SimAb-style models.
when_to_use_negative:
- The question is how dust grain fraction or planetesimal ratio (not initial orbital
  distance) sets C/O — the paper shows dust-dominated planets approach solar C/O while
  planetesimal-dominated planets approach the solid-phase C/O, requiring a different
  sweep
- The question is metallicity (subsolar vs supersolar) rather than C/O ratio — for
  that, planetesimal accretion amount must be varied alongside orbital distance
- Randomized sampling of initial conditions is desired — use Mrun.py multi-run mode
  instead, which randomly assigns initial orbital distance, core mass, planetesimal
  ratio, and dust grain fraction
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: SimAb
  role: Planet formation simulation whose gas/solid-phase abundance model across the
    H2O and CO ice lines produces the C/O ratio dependence on initial orbital distance
  repo: github.com/nkhorshid/simab
- name: Srun.py
  role: 'Single-simulation entry point: run once per initial-orbital-distance grid
    point with fixed core mass, planetesimal ratio, and dust grain fraction'
  repo: github.com/nkhorshid/simab
- name: Jupyter
  role: Plotting the resulting C/O ratio versus initial orbital distance from the
    collected run_sum(n).txt outputs
  repo: github.com/nkhorshid/simab
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1051/0004-6361/202141455
    title: 'SimAb: A simple, fast, and flexible model to assess the effects of planet
      formation on the atmospheric composition of gas giants'
  source_content_hash: sha256:2253d94f6bf688adee7aa97a6232b17bca938dfa6d69ce8cabdd310a47eab07c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - SimAb
  - Srun.py
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# ice-line-crossing-abundance-analysis

## Summary

A method for quantifying how a protoplanet's initial orbital distance relative to the H2O and CO ice lines controls the final atmospheric C/O ratio, using the SimAb planet formation model's run_sum(n).txt abundance outputs.

## When to use

Use when studying how initial formation location (inside the H2O ice line, between H2O and CO ice lines, or beyond the CO ice line) determines the final C/O ratio of a gas giant's atmosphere in SimAb-style models.

## When NOT to use

- The question is how dust grain fraction or planetesimal ratio (not initial orbital distance) sets C/O — the paper shows dust-dominated planets approach solar C/O while planetesimal-dominated planets approach the solid-phase C/O, requiring a different sweep
- The question is metallicity (subsolar vs supersolar) rather than C/O ratio — for that, planetesimal accretion amount must be varied alongside orbital distance
- Randomized sampling of initial conditions is desired — use Mrun.py multi-run mode instead, which randomly assigns initial orbital distance, core mass, planetesimal ratio, and dust grain fraction

## Inputs

- SimAb parameter values used for the paper's figures, available in the repo's output folder (core mass, planetesimal ratio, dust grain fraction to fix)
- Grid of initial orbital distances spanning inside the H2O ice line, between the H2O and CO ice lines, and beyond the CO ice line
- Mature planet mass and final orbital distance for Srun.py single-simulation mode

## Outputs

- run_sum(n).txt per run, containing final composition as [(x/H) of the planet]/[(X/H) of the sun]
- co_ratio_vs_initial_orbital_distance.csv with one final C/O ratio per orbital-distance grid point

## How to apply

Fix the core mass, planetesimal ratio, and dust grain fraction using the SimAb parameter values stored in the repo's output folder (the values used for the paper's figures). Define a grid of initial orbital distances explicitly spanning the three regimes: inside the H2O ice line, between the H2O and CO ice lines, and beyond the CO ice line, since the gas-phase and solid-phase C/O ratios differ across these ice lines. Run each grid point through SimAb via Srun.py (single simulation mode with specified initial conditions plus mature planet mass and orbital distance), collecting the final composition from run_sum(n).txt, which records [(x/H) of the planet]/[(X/H) of the sun]. Convert these elemental abundance ratios into a final atmospheric C/O ratio per orbital distance, and write the C/O-versus-distance table to co_ratio_vs_initial_orbital_distance.csv. The expected physical behavior is that planets initiating formation farther than the CO ice line have a C/O ratio closer to the solar value, while distances inside the ice lines yield C/O set by the local solid-phase composition.

## Related tools

- **SimAb** (Planet formation simulation whose gas/solid-phase abundance model across the H2O and CO ice lines produces the C/O ratio dependence on initial orbital distance) — github.com/nkhorshid/simab
- **Srun.py** (Single-simulation entry point: run once per initial-orbital-distance grid point with fixed core mass, planetesimal ratio, and dust grain fraction) — github.com/nkhorshid/simab
- **Jupyter** (Plotting the resulting C/O ratio versus initial orbital distance from the collected run_sum(n).txt outputs) — github.com/nkhorshid/simab

## Examples

```
python Srun.py  # with initial conditions: core mass, initial orbital distance (grid point), planetesimal ratio, dust grain fraction, plus mature planet mass and orbital distance
```

## Evaluation signals

- co_ratio_vs_initial_orbital_distance.csv contains exactly one C/O value per orbital-distance grid point
- Each run produces a run(n) folder with run_sum(n).txt recording composition as [(x/H)_planet]/[(X/H)_sun]
- Grid points beyond the CO ice line yield C/O ratios closer to the solar value, reproducing the paper's reported dependence
- C/O values differ across the H2O and CO ice-line boundaries, reflecting the differing gas-phase and solid-phase C/O at different orbital distances
- Core mass, planetesimal ratio, and dust grain fraction are identical across all sweep runs, isolating orbital distance as the varied parameter

## Limitations

- Results are model-specific: the paper shows the initial protoplanet core mass does not influence the final atmospheric composition in the context of the SimAb model, so conclusions may not transfer to other formation models
- The sweep isolates orbital distance only; initial orbital distance and accreted planetesimals together determine subsolar or supersolar metallicity, so C/O alone does not fully constrain formation conditions
- The number of random runs and paper parameter values matter: the author used 130,000 runs for the paper and the paper's parameter values are only available in the repo's output folder

## Evidence

- [abstract] initial orbital distance determines C/O across ice lines: "The initial orbital distance affects the C/O ratio due to the different C/O ratios in the gas phase and the solid phase at different orbital distances."
- [abstract] beyond CO ice line solar C/O: "Supersolar metallicity planets that initiate their formation farther than the CO ice line have a C/O ratio closer to the solar value."
- [readme] Srun.py single-run usage: "The single simulation mode that is run through running Srun.py, by specifying the initial conditions (the core mass, the initial orbital distance, the planetesimal ratio, and the dust grain fraction)"
- [readme] run_sum(n).txt composition format: "The final composition is recorded in a file called run_sum(n).txt as [(x/H)of the planet]/[(X/H) of the sun]."
- [readme] paper parameter values in output folder: "The values used to write the SimAb paper are available in the output folder."
