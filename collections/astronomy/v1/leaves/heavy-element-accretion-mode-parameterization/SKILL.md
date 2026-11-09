---
name: heavy-element-accretion-mode-parameterization
description: Decision support for interpreting a formed giant planet's atmospheric
  C/O ratio in terms of the dominant source of accreted heavy elements (dust grains
  vs. planetesimals).
when_to_use_negative:
- When the question concerns the influence of initial protoplanet core mass on atmospheric
  composition — the paper shows the initial core mass does not influence final composition
- When subsolar vs. supersolar metallicity (not C/O accretion-mode signatures) is
  the target diagnostic, where planetesimal accretion amount and initial orbital distance
  relative to ice lines dominate
- When a detailed N-body or disk-evolution model is needed — SimAb is explicitly a
  basic, fast, flexible model with stated simplifying assumptions
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: SimAb
  role: Planet formation simulation whose outputs (run_sum(n).txt, test(m).txt) provide
    the atmospheric C/O ratios for the dust- vs. planetesimal-dominated comparison
  repo: github.com/nkhorshid/simab
- name: Srun.py
  role: Single simulation mode used to run the paired dust-grain-dominated and planetesimal-dominated
    configurations with user-specified initial conditions
  repo: github.com/nkhorshid/simab
- name: Mrun.py
  role: Multi run mode that randomly assigns initial orbital distance, initial core
    mass, initial planetesimal ratio, and initial dust grain fraction to extend the
    comparison to ensembles
  repo: github.com/nkhorshid/simab
- name: Jupyter
  role: Used to plot the simulation results
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202141455
    title: 'SimAb: A simple, fast, and flexible model to assess the effects of planet
      formation on the atmospheric composition of gas giants'
  source_content_hash: sha256:0ac3a56f471946a5811ba8d46bb88a4e4b9ad03d12d8cc24a0744c122b207288
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
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# heavy-element-accretion-mode-parameterization

## Summary

A parameterization skill for SimAb that contrasts the two dominant modes of accreted heavy elements — dust grain fraction versus planetesimal ratio — to determine whether a gas giant's final atmospheric C/O ratio tracks the solar value or the disk solid phase.

## When to use

Decision support for interpreting a formed giant planet's atmospheric C/O ratio in terms of the dominant source of accreted heavy elements (dust grains vs. planetesimals).

## When NOT to use

- When the question concerns the influence of initial protoplanet core mass on atmospheric composition — the paper shows the initial core mass does not influence final composition
- When subsolar vs. supersolar metallicity (not C/O accretion-mode signatures) is the target diagnostic, where planetesimal accretion amount and initial orbital distance relative to ice lines dominate
- When a detailed N-body or disk-evolution model is needed — SimAb is explicitly a basic, fast, flexible model with stated simplifying assumptions

## Inputs

- Initial conditions for Srun.py: core mass, initial orbital distance, planetesimal ratio, dust grain fraction
- Mature planet mass and final orbital distance
- For Mrun.py: mature planet mass, final orbital distance, number of random runs, output folder name
- Pre-created run(n) output folders

## Outputs

- run_sum(n).txt with final composition as [(x/H) of the planet]/[(X/H) of the sun]
- Per-run test(m).txt files in run(n) folders
- Comparison table (co_ratio_comparison.csv) of final C/O ratios for dust-dominated vs. planetesimal-dominated accretion
- Optional Jupyter plots of the results

## How to apply

Set up a dust-grain-dominated pair using Srun.py: specify initial conditions (core mass, initial orbital distance, planetesimal ratio, dust grain fraction) so most heavy elements arrive as dust grains, plus the mature planet mass and orbital distance. Build the planetesimal-dominated counterpart with planetesimal ratio near unity and minimal dust grain fraction at the same mature planet mass and orbital distance. Optionally extend to randomized ensembles with Mrun.py, which randomly assigns initial orbital distance, initial core mass, initial planetesimal ratio, and initial dust grain fraction (130,000 runs were used for the paper), creating a run(n) folder per run. Parse run_sum(n).txt, which records final composition as [(x/H) of the planet]/[(X/H) of the sun], and the per-run test(m).txt files. Compute the final C/O ratio for each configuration and assemble a comparison table (e.g., co_ratio_comparison.csv) contrasting the two accretion modes.

## Related tools

- **SimAb** (Planet formation simulation whose outputs (run_sum(n).txt, test(m).txt) provide the atmospheric C/O ratios for the dust- vs. planetesimal-dominated comparison) — github.com/nkhorshid/simab
- **Srun.py** (Single simulation mode used to run the paired dust-grain-dominated and planetesimal-dominated configurations with user-specified initial conditions) — github.com/nkhorshid/simab
- **Mrun.py** (Multi run mode that randomly assigns initial orbital distance, initial core mass, initial planetesimal ratio, and initial dust grain fraction to extend the comparison to ensembles) — github.com/nkhorshid/simab
- **Jupyter** (Used to plot the simulation results)

## Examples

```
python Srun.py  # specify core mass, initial orbital distance, planetesimal ratio, dust grain fraction, and mature planet mass/orbital distance
```

## Evaluation signals

- Dust-grain-dominated configurations yield a final C/O ratio close to the solar C/O ratio in the comparison table
- Planetesimal-dominated configurations yield a final C/O ratio approaching the disk solid-phase C/O ratio
- run_sum(n).txt exists for each run and compositions are normalized as [(x/H) of the planet]/[(X/H) of the sun]
- Each run folder run(n) contains the expected per-run test(m).txt output files matching the requested number of random runs
- Paired configurations share the same mature planet mass and orbital distance, differing only in planetesimal ratio and dust grain fraction

## Limitations

- SimAb is a basic, fast, and flexible model; its simplifying assumptions are documented in the SimAb paper (arXiv:2111.00279)
- In Mrun.py mode the user must manually create a folder named run(n) before running; output naming (test(m).txt, run_sum(n).txt) depends on this convention
- Initial orbital distance confounds the C/O signal because gas-phase and solid-phase C/O ratios differ with orbital distance, so accretion-mode comparisons must control for initial orbital distance

## Evidence

- [abstract] Dust vs. planetesimal C/O signatures: "Planets that accrete most of their heavy elements through dust grains will have a C/O ratio close to the solar C/O ratio"
- [abstract] Orbital distance effect on C/O: "The initial orbital distance affects the C/O ratio due to the different C/O ratios in the gas phase and the solid phase at different orbital distances."
- [readme] Srun.py usage: "The single simulation mode that is run through running Srun.py, by specifying the initial conditions (the core mass, the initial orbital distance, the planetesimal ratio"
- [readme] Mrun.py randomization and run folders: "you can specify the mass and the final orbital distance of the mature planet and the simulation randomly asigns initial orbital distance, initial core mass"
- [readme] Output format: "The final composition is recorded in a file called run_sum(n).txt as [(x/H)of the planet]/[(X/H) of the sun]."
