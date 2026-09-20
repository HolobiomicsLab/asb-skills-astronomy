---
name: randomized-initial-condition-sampling
description: Use when you need to map how randomized initial conditions (initial orbital
  distance, core mass, planetesimal ratio, dust grain fraction) determine a giant
  planet's final atmospheric metallicity (subsolar vs supersolar, ratio <1 vs >1)
  or C/O ratio in SimAb.
when_to_use_negative:
- When you want a single simulation with user-specified, fixed initial conditions
  — use Srun.py (single simulation mode) instead of randomized multi-run sampling.
- When the question concerns parameters SimAb does not sample randomly (e.g. detailed
  growth histories or N-body dynamics) — Mrun.py only randomizes initial orbital distance,
  initial core mass, initial planetesimal ratio, and initial dust grain fraction.
- When the target quantity is something the model shows is insensitive to initial
  conditions — e.g. the paper shows the initial protoplanet core mass does not influence
  the final atmospheric composition.
edam_operation: http://edamontology.org/operation_2945
tools:
- name: Mrun.py
  role: 'Multi-run mode entry point: randomly assigns initial conditions per run for
    a fixed mature planet mass and final orbital distance, writing test(m).txt and
    run_sum(n).txt outputs'
  repo: github.com/nkhorshid/simab
- name: SimAb
  role: The planet formation and atmosphere accretion model whose parameter space
    is being sampled; the paper's own input values are available in the output folder
  repo: github.com/nkhorshid/simab
- name: Jupyter
  role: Plotting the multi-run results (e.g. metallicity class vs initial orbital
    distance and accreted planetesimals)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202141455
    title: 'SimAb: A simple, fast, and flexible model to assess the effects of planet
      formation on the atmospheric composition of gas giants'
  source_content_hash: sha256:6db3a5eb8c9018eef4cbf480839d01e9f3db037bc2e8edb27486fe40ba9289de
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - Mrun.py
  - SimAb
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# randomized-initial-condition-sampling

## Summary

A Monte Carlo parameter-space exploration using SimAb's Mrun.py multi-run mode, which randomly samples initial conditions (initial orbital distance, initial core mass, initial planetesimal ratio, initial dust grain fraction) for a fixed mature planet mass and final orbital distance. It reveals which initial conditions jointly control subsolar vs supersolar final atmospheric metallicity and C/O ratios.

## When to use

Use when you need to map how randomized initial conditions (initial orbital distance, core mass, planetesimal ratio, dust grain fraction) determine a giant planet's final atmospheric metallicity (subsolar vs supersolar, ratio <1 vs >1) or C/O ratio in SimAb.

## When NOT to use

- When you want a single simulation with user-specified, fixed initial conditions — use Srun.py (single simulation mode) instead of randomized multi-run sampling.
- When the question concerns parameters SimAb does not sample randomly (e.g. detailed growth histories or N-body dynamics) — Mrun.py only randomizes initial orbital distance, initial core mass, initial planetesimal ratio, and initial dust grain fraction.
- When the target quantity is something the model shows is insensitive to initial conditions — e.g. the paper shows the initial protoplanet core mass does not influence the final atmospheric composition.

## Inputs

- Mature planet mass (fixed across all runs)
- Final orbital distance of the mature planet (fixed across all runs)
- Number of random runs (paper: 130,000; reduced counts acceptable)
- Output folder name
- Pre-created folders named run(n), one per run batch

## Outputs

- Per-run output files test(m).txt inside each run(n) folder (one per random test)
- run_sum(n).txt files recording final composition as [(x/H) of the planet]/[(X/H) of the sun]
- Aggregated classification per run: metallicity class (subsolar if ratio < 1, supersolar if ratio > 1), initial orbital distance, and accreted planetesimal amount
- Plots (via the provided Jupyter codes) of metallicity/C/O vs initial conditions

## How to apply

Run Mrun.py (the multi-run mode) specifying the mature planet's mass, its final orbital distance, the number of random runs, and an output folder; the paper used 130,000 runs, but a reduced count is acceptable when compute-limited. Before running, create a folder named run(n) for each run batch, where n is the run number. Mrun.py randomly assigns the initial orbital distance, initial core mass, initial planetesimal ratio, and initial dust grain fraction per random test, recording each single run as test(m).txt inside run(n). The final composition is written to run_sum(n).txt as [(x/H) of the planet]/[(X/H) of the sun]; parse these files with Python to classify each run as subsolar (ratio < 1) or supersolar (ratio > 1) and aggregate the initial orbital distance and accreted planetesimal amount per run into a CSV. Rationale: the paper shows the initial orbital distance together with accreted planetesimals determines subsolar vs supersolar metallicity, with planetesimals the main source of supersolar metallicity, so the randomized sweep is what exposes that joint dependence.

## Related tools

- **Mrun.py** (Multi-run mode entry point: randomly assigns initial conditions per run for a fixed mature planet mass and final orbital distance, writing test(m).txt and run_sum(n).txt outputs) — github.com/nkhorshid/simab
- **SimAb** (The planet formation and atmosphere accretion model whose parameter space is being sampled; the paper's own input values are available in the output folder) — github.com/nkhorshid/simab
- **Jupyter** (Plotting the multi-run results (e.g. metallicity class vs initial orbital distance and accreted planetesimals))

## Examples

```
python Mrun.py  # specify mature planet mass, final orbital distance, number of random runs, and output folder; create run(n) folders first
```

## Evaluation signals

- Each run(n) folder contains per-test test(m).txt files and a run_sum(n).txt whose metallicity is expressed as [(x/H) of the planet]/[(X/H) of the sun].
- The mature planet mass and final orbital distance are identical across all runs; only the four randomly assigned initial parameters vary.
- Classifying runs as subsolar (ratio < 1) or supersolar (ratio > 1) reproduces the paper's finding that initial orbital distance plus accreted planetesimals jointly determine the outcome.
- Supersolar-metallicity runs are traceable primarily to large accreted planetesimal amounts, matching 'planetesimals are the main source for reaching supersolar metallicity planets'.
- With a reduced run count, the sampled distribution of the four random parameters still covers the same parameter space as the paper's 130,000-run sweep.

## Limitations

- The initial protoplanet core mass does not influence the final atmospheric composition in the model, so that sampled dimension contributes no explanatory signal.
- The paper's statistical basis is 130,000 random runs; reduced run counts for compute-limited execution lower confidence in parameter-space coverage.
- Results are valid only within SimAb's parametrized assumptions; each random test requires a pre-created run(n) folder or outputs cannot be recorded.

## Evidence

- [readme] randomly assigned initial conditions and file layout: "you can specify the mass and the final orbital distance of the mature planet and the simulation randomly asigns initial orbital distance, initial core mass, initial planetesimal ratio, and initial"
- [readme] output file format: "The final composition is recorded in a file called run_sum(n).txt as [(x/H)of the planet]/[(X/H) of the sun]."
- [readme] run count and folder setup: "You need to also indicate the number of the random runs (I used 130,000 runs for the paper) and the folder name where the output is recorded."
- [abstract] finding on joint control of metallicity: "the initial orbital distance together with the amount of accreted planetesimals cause the planet to have subsolar or supersolar metallicity"
- [abstract] finding on planetesimal source: "We show that planetesimals are the main source for reaching supersolar metallicity planets."
