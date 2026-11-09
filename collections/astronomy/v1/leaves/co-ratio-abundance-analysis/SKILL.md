---
name: co-ratio-abundance-analysis
description: Use when you have SimAb run outputs (run_sum(n).txt files with [(x/H)
  of the planet]/[(X/H) of the sun]) and need to classify subsolar vs supersolar metallicity
  and compute C/O ratios to trace the source of heavy elements.
when_to_use_negative:
- When only a single simulation with fully user-specified initial conditions is needed
  — use the Srun.py single-run mode instead of a multi-run composition analysis.
- When the question is whether the initial core mass matters — the paper shows the
  initial protoplanet core mass does not influence the final atmospheric composition,
  so core mass is not a useful discriminator here.
- When inputs are raw observables (e.g., spectra) rather than SimAb composition outputs
  — this skill operates on SimAb's (x/H) ratio files, not on atmospheric retrieval
  data.
edam_operation: http://edamontology.org/operation_2945
tools:
- name: SimAb
  role: Planet-formation simulation whose run outputs supply the (x/H) ratios from
    which C/O and metallicity are computed
  repo: github.com/nkhorshid/simab
- name: Mrun.py
  role: Multi-run driver that randomly assigns initial orbital distance, core mass,
    planetesimal ratio, and dust grain fraction, and writes run_sum(n).txt composition
    files
  repo: github.com/nkhorshid/simab
- name: Jupyter
  role: Plotting of resulting C/O ratio and metallicity distributions
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202141455
    title: 'SimAb: A simple, fast, and flexible model to assess the effects of planet
      formation on the atmospheric composition of gas giants'
  source_content_hash: sha256:4a1843836ac004c1475981c771a3f711a92aa35a2931040bbfb632529fd959c9
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
yes_rate: 0.5
n_claims: 8
---

# co-ratio-abundance-analysis

## Summary

Compute and interpret elemental abundance ratios (C/O and metallicity relative to solar) from SimAb planet-formation outputs, linking dust-grain vs planetesimal accretion to supersolar metallicity and near-solar C/O. This is the paper's core diagnostic for constraining where and from what material giant planets formed.

## When to use

Use when you have SimAb run outputs (run_sum(n).txt files with [(x/H) of the planet]/[(X/H) of the sun]) and need to classify subsolar vs supersolar metallicity and compute C/O ratios to trace the source of heavy elements.

## When NOT to use

- When only a single simulation with fully user-specified initial conditions is needed — use the Srun.py single-run mode instead of a multi-run composition analysis.
- When the question is whether the initial core mass matters — the paper shows the initial protoplanet core mass does not influence the final atmospheric composition, so core mass is not a useful discriminator here.
- When inputs are raw observables (e.g., spectra) rather than SimAb composition outputs — this skill operates on SimAb's (x/H) ratio files, not on atmospheric retrieval data.

## Inputs

- run_sum(n).txt files from SimAb multi-run mode, each storing final composition as [(x/H) of the planet]/[(X/H) of the sun]
- Per-run metadata from run(n)/test(m).txt folders: initial orbital distance, initial core mass, initial planetesimal ratio, initial dust grain fraction
- Solar reference abundances used as the denominator in the ratio

## Outputs

- Per-run metallicity relative to solar (subsolar < 1 < supersolar classification)
- Per-run C/O ratio of the mature planet's atmosphere
- Aggregated CSV/JSON record with metallicity class, C/O, initial orbital distance, and accreted planetesimal amount per run
- Inferred boundaries (lower and upper) on the initial orbital distance of supersolar metallicity planets

## How to apply

Parse each run_sum(n).txt produced by Mrun.py, which stores final composition as [(x/H) of the planet]/[(X/H) of the sun], then compute metallicity relative to solar and the C/O ratio per run. Classify runs as subsolar (ratio < 1) or supersolar (ratio > 1) and record each run's initial orbital distance, planetesimal ratio, and dust grain fraction. Interpret C/O via the paper's mechanism: initial orbital distance changes C/O because gas-phase and solid-phase C/O differ across orbital distances, while accretion source sets the ratio — dust-dominated planets end near the solar C/O ratio, planetesimal-dominated planets approach the solid-phase C/O. For supersolar planets, use C/O and metallicity together to bound the initial orbital distance, checking whether formation began farther than the CO ice line (such planets have C/O closer to the solar value). Aggregate per-run metallicity and C/O values into a structured CSV/JSON record for comparison against the paper's findings.

## Related tools

- **SimAb** (Planet-formation simulation whose run outputs supply the (x/H) ratios from which C/O and metallicity are computed) — github.com/nkhorshid/simab
- **Mrun.py** (Multi-run driver that randomly assigns initial orbital distance, core mass, planetesimal ratio, and dust grain fraction, and writes run_sum(n).txt composition files) — github.com/nkhorshid/simab
- **Jupyter** (Plotting of resulting C/O ratio and metallicity distributions)

## Examples

```
python Mrun.py  # then: import glob; [parse(run_sum) for run_sum in glob.glob('run*/run_sum(*).txt')]  # [(x/H)planet]/[(X/H)sun]
```

## Evaluation signals

- Reproduces the paper's finding that planetesimals are the main source for reaching supersolar metallicity, while dust-dominated planets show a more solar composition.
- Supersolar-metallicity runs that initiate formation farther than the CO ice line show a C/O ratio closer to the solar value.
- Subsolar/supersolar classification co-varies with initial orbital distance and accreted planetesimal amount, as reported in the paper.
- Every parsed run_sum(n).txt yields numeric (x/H) ratios in planet/solar form; missing run folders or absent run_sum files are surfaced as parse failures, not silently dropped runs.
- Using C/O and metallicity together yields lower and upper bounds on the initial orbital distance of supersolar planets, consistent with the paper's constraint.

## Limitations

- The paper used 130,000 random runs; reduced run counts (if compute-limited) lower the statistical power of the subsolar/supersolar classification.
- Results are model-dependent on SimAb's assumptions; the paper's values are only available in the output folder for reproducing the paper's configuration.
- The initial protoplanet core mass does not influence final atmospheric composition in this model, so it cannot be used to explain abundance differences.

## Evidence

- [readme] The final composition is recorded in a file called run_sum(n).txt as [(x/H)of the planet]/[(X/H) of the sun].: "The final composition is recorded in a file called run_sum(n).txt as [(x/H)of the planet]/[(X/H) of the sun]."
- [abstract] The initial orbital distance affects the C/O ratio due to the different C/O ratios in the gas phase and the solid phase at different orbital distances.: "The initial orbital distance affects the C/O ratio due to the different C/O ratios in the gas phase and the solid phase at different orbital distances."
- [abstract] We show that planetesimals are the main source for reaching supersolar metallicity planets.: "We show that planetesimals are the main source for reaching supersolar metallicity planets."
- [abstract] Supersolar metallicity planets that initiate their formation farther than the CO ice line have a C/O ratio closer to the solar value.: "Supersolar metallicity planets that initiate their formation farther than the CO ice line have a C/O ratio closer to the solar value."
- [abstract] By using the C/O ratio and metallicity together we can put a lower and upper boundary on the initial orbital distance where supersolar metallicity planets are formed.: "By using the C/O ratio and metallicity together we can put a lower and upper boundary on the initial orbital distance where supersolar metallicity planets are formed."
