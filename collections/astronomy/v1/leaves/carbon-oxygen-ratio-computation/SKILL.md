---
name: carbon-oxygen-ratio-computation
description: Use when interpreting or predicting a giant planet's atmospheric C/O
  ratio as a tracer of its heavy-element accretion pathway (dust grains vs planetesimals),
  e.g. when comparing SimAb run_sum(n).txt outputs or planning paired formation scenarios.
when_to_use_negative:
- The analysis goal is metallicity (subsolar vs supersolar) alone rather than the
  C/O signature — planetesimals are the main source for reaching supersolar metallicity,
  so a C/O-only contrast does not address that question.
- The parameter of interest is the initial core mass — the paper shows it does not
  influence the final atmospheric composition, so sweeping core mass yields no C/O
  signal.
- The target is not a gas giant whose accreted atmosphere is being modeled — SimAb
  focuses on atmosphere accretion of gas giant planets.
edam_operation: http://edamontology.org/operation_2426
tools:
- name: SimAb
  role: Planet formation simulation whose atmosphere-accretion runs supply the solar-normalized
    elemental abundances from which the final C/O ratio is computed
  repo: github.com/nkhorshid/simab
- name: Srun.py
  role: Single-run mode used to execute the paired dust-grain-dominated vs planetesimal-dominated
    configurations with user-specified initial conditions
  repo: github.com/nkhorshid/simab
- name: Mrun.py
  role: Multi-run mode that randomizes initial core mass, orbital distance, planetesimal
    ratio, and dust grain fraction to build the C/O distribution for a fixed mature
    planet mass and final orbital distance
  repo: github.com/nkhorshid/simab
- name: Jupyter
  role: Used to plot the run outputs (e.g. C/O versus dust grain fraction and planetesimal
    ratio) after parsing run_sum(n).txt
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1051/0004-6361/202141455
    title: 'SimAb: A simple, fast, and flexible model to assess the effects of planet
      formation on the atmospheric composition of gas giants'
  source_content_hash: sha256:584710e807e1263bd9ed81291ddd3005e3e3c356d3bcb6c080796e43c784e904
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - Mrun.py
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

# carbon-oxygen-ratio-computation

## Summary

Computes the final atmospheric C/O ratio of a formed gas giant from SimAb formation runs (Srun.py/Mrun.py) and contrasts dust-grain-dominated versus planetesimal-dominated heavy-element accretion, whose C/O signatures differ (near-solar vs solid-phase). It turns the run_sum(n).txt solar-normalized [(x/H) of the planet]/[(X/H) of the sun] outputs into a per-configuration C/O comparison.

## When to use

Use when interpreting or predicting a giant planet's atmospheric C/O ratio as a tracer of its heavy-element accretion pathway (dust grains vs planetesimals), e.g. when comparing SimAb run_sum(n).txt outputs or planning paired formation scenarios.

## When NOT to use

- The analysis goal is metallicity (subsolar vs supersolar) alone rather than the C/O signature — planetesimals are the main source for reaching supersolar metallicity, so a C/O-only contrast does not address that question.
- The parameter of interest is the initial core mass — the paper shows it does not influence the final atmospheric composition, so sweeping core mass yields no C/O signal.
- The target is not a gas giant whose accreted atmosphere is being modeled — SimAb focuses on atmosphere accretion of gas giant planets.

## Inputs

- Initial conditions for Srun.py: core mass, initial orbital distance, planetesimal ratio, dust grain fraction, plus the mature planet mass and orbital distance
- Mrun.py specification: mature planet mass, final orbital distance, number of random runs, output folder name run(n)
- SimAb run outputs: run_sum(n).txt solar-normalized composition files and per-run test(m).txt files in the run(n) folders

## Outputs

- Final C/O ratio per configuration, derived from the normalized carbon (C/H) and oxygen (O/H) entries of run_sum(n).txt
- co_ratio_comparison.csv: comparison table contrasting dust-grain-dominated versus planetesimal-dominated accretion C/O ratios
- Plots of the C/O results produced with the available Jupyter codes

## How to apply

Run paired single simulations with Srun.py, holding the mature planet mass and orbital distance fixed while flipping the heavy-element source: a dust-grain-dominated case (high dust grain fraction, planetesimal ratio near zero) and a planetesimal-dominated case (planetesimal ratio near unity, minimal dust grain fraction), each also specifying initial core mass and initial orbital distance. Optionally extend the pair with Mrun.py, which randomly assigns initial orbital distance, initial core mass, initial planetesimal ratio, and initial dust grain fraction for a chosen mature planet mass, final orbital distance, and number of random runs (130,000 were used in the paper); pre-create a run(n) folder per batch, with per-run outputs in test(m).txt. Parse run_sum(n).txt, where the final composition is recorded as [(x/H) of the planet]/[(X/H) of the sun], and derive each C/O ratio from the normalized carbon and oxygen entries. Assemble the per-configuration values into co_ratio_comparison.csv contrasting dust-dominated versus planetesimal-dominated accretion, plotting trends with the available Jupyter codes. Validate against the paper's signature: dust-grain-dominated planets land close to the solar C/O ratio while planetesimal-dominated planets approach the disk solid-phase C/O ratio, which itself varies with initial orbital distance because gas-phase and solid-phase C/O differ with distance.

## Related tools

- **SimAb** (Planet formation simulation whose atmosphere-accretion runs supply the solar-normalized elemental abundances from which the final C/O ratio is computed) — github.com/nkhorshid/simab
- **Srun.py** (Single-run mode used to execute the paired dust-grain-dominated vs planetesimal-dominated configurations with user-specified initial conditions) — github.com/nkhorshid/simab
- **Mrun.py** (Multi-run mode that randomizes initial core mass, orbital distance, planetesimal ratio, and dust grain fraction to build the C/O distribution for a fixed mature planet mass and final orbital distance) — github.com/nkhorshid/simab
- **Jupyter** (Used to plot the run outputs (e.g. C/O versus dust grain fraction and planetesimal ratio) after parsing run_sum(n).txt)

## Examples

```
python Srun.py  # specify core mass, initial orbital distance, planetesimal ratio, dust grain fraction, and mature planet mass & orbital distance
```

## Evaluation signals

- Dust-grain-dominated rows of co_ratio_comparison.csv show a final C/O ratio close to the solar C/O ratio.
- Planetesimal-dominated rows (planetesimal ratio near unity) shift the final C/O toward the disk solid-phase value at the chosen initial orbital distance.
- run_sum(n).txt entries parse cleanly as [(x/H) of the planet]/[(X/H) of the sun] (solar-normalized ratios) before C/O is computed.
- Changing only the initial core mass leaves the recorded composition unchanged, matching the model's core-mass invariance.
- Supersolar-metallicity runs initiated farther than the CO ice line return C/O ratios closer to the solar value.

## Limitations

- The initial protoplanet core mass does not influence the final atmospheric composition in this model, so C/O cannot constrain core mass.
- C/O alone is degenerate with initial orbital distance because gas-phase and solid-phase C/O ratios differ with distance; only C/O plus metallicity together bound the initial orbital distance of supersolar-metallicity planets.
- Mrun.py requires manually creating the run(n) output folders beforehand and choosing the number of random runs (130,000 in the paper); per-run outputs accumulate as test(m).txt files.
- Scope is limited to atmosphere accretion of gas giant planets as modeled by the basic, fast SimAb model.

## Evidence

- [abstract] Planets that accrete most of their heavy elements through dust grains will have a C/O ratio close to the solar C/O ratio: "Planets that accrete most of their heavy elements through dust grains will have a C/O ratio close to the solar C/O ratio"
- [abstract] The initial orbital distance affects the C/O ratio due to the different C/O ratios in the gas phase and the solid phase at different orbital distances.: "The initial orbital distance affects the C/O ratio due to the different C/O ratios in the gas phase and the solid phase at different orbital distances."
- [intro] The single simulation mode that is run through running Srun.py, by specifying the initial conditions: "The single simulation mode that is run through running Srun.py, by specifying the initial conditions"
- [readme] The final composition is recorded in a file called run_sum(n).txt as [(x/H)of the planet]/[(X/H) of the sun].: "The final composition is recorded in a file called run_sum(n).txt as [(x/H)of the planet]/[(X/H) of the sun]."
- [readme] the simulation randomly asigns initial orbital distance, initial core mass, initial planetesimal ratio, and initial dust grain fraction: "the simulation randomly asigns initial orbital distance, initial core mass, initial planetesimal ratio, and initial dust grain fraction"
- [abstract] We show that the initial protoplanet core mass does not influence the final composition of the planetary atmosphere in the context of our model.: "We show that the initial protoplanet core mass does not influence the final composition of the planetary atmosphere in the context of our model."
