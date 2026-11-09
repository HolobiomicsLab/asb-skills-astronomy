---
name: simulation-results-table-generation
description: Use when you have standardized outputs from an ensemble simulation (e.g.,
  OSPREI '.ens' runs for CMEs) and need to tabulate which input-output pairs exceed
  a correlation coefficient cutoff, producing per-case correlation tables.
when_to_use_negative:
- Input is a single deterministic simulation (nRuns = 1) with no varied parameters
  — no ensemble correlations can be computed
- The goal is a best-fit parameter tuning rather than uncertainty/exploration of input-output
  relationships; OSPREI's proof-of-concept deliberately avoids fine-tuning inputs
tools:
- name: OSPREI
  role: Generates the ensemble simulation outputs; processOSPREI.py loads them into
    a results object and produces the ensemble correlation figures
  repo: https://github.com/ckay314/OSPREI
- name: ForeCAT
  role: Provides the coronal trajectory outputs (final latitude, longitude, tilt)
    that serve as correlated outputs in the table
- name: ANTEATR
  role: Provides interplanetary propagation outputs (arrival time, speed, size, shape)
    used as correlated outputs
- name: FIDO
  role: Provides synthetic in situ magnetic field and velocity profiles used as correlated
    outputs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1029/2021sw002914
    title: 'OSPREI: A Coupled Approach to Modeling CME‐Driven Space Weather With Automatically
      Generated, User‐Friendly Outputs'
  source_content_hash: sha256:5f129c4b5bc07ab06a02727d6cd92a17e28375321b123e0abd6b9799f52606ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ckay314/OSPREI
  tools:
  - ANTEATR
  - FIDO
  - ForeCAT
  - OSPREI
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# simulation-results-table-generation

## Summary

Generates structured correlation tables from simulation ensemble results by computing correlation coefficients between varied input parameters and quantifiable outputs, then filtering pairs by a correlation-magnitude cutoff (0.5) as used in OSPREI's Figure 9 ensemble trend analysis.

## When to use

Use when you have standardized outputs from an ensemble simulation (e.g., OSPREI '.ens' runs for CMEs) and need to tabulate which input-output pairs exceed a correlation coefficient cutoff, producing per-case correlation tables.

## When NOT to use

- Input is a single deterministic simulation (nRuns = 1) with no varied parameters — no ensemble correlations can be computed
- The goal is a best-fit parameter tuning rather than uncertainty/exploration of input-output relationships; OSPREI's proof-of-concept deliberately avoids fine-tuning inputs

## Inputs

- Standardized OSPREI ensemble output files (results from runs where nRuns > 1)
- '.ens' ensemble configuration file defining varied parameters and their ranges
- List of quantifiable outputs (final latitude, longitude, tilt, arrival time, speed, in situ properties)

## Outputs

- Per-CME correlation table of input-output pairs exceeding the 0.5 correlation-magnitude cutoff (6 inputs × 11 outputs per CME)
- Correlation coefficient values in [-1, 1] for each retained pair

## How to apply

Load the standardized output files from the ensemble runs (e.g., the 22 April and 09 May 2021 CMEs) into a results object using processOSPREI.py, reading the varied-parameter ranges from the '.ens' ensemble configuration files referenced by nRuns > 1. For each CME, compute the correlation coefficient between every varied ensemble input parameter (e.g., CMElat, CMElon, CMEtilt, CMEvr, CMEAW, flux-rope parameters) and every quantifiable output (final latitude, longitude, tilt, arrival time, speed, in situ properties). Apply a 0.5 correlation-magnitude cutoff to retain only strong input-output pairs, then assemble a per-CME table; for OSPREI this yields 6 varied inputs by 11 outputs exceeding the cutoff per CME (not identical parameters for each CME). Validate that coefficients lie in [-1, 1] and that the table structure matches, checking e.g. that CME 1 shows strong correlations of final latitude, longitude, and tilt with their initial values.

## Related tools

- **OSPREI** (Generates the ensemble simulation outputs; processOSPREI.py loads them into a results object and produces the ensemble correlation figures) — https://github.com/ckay314/OSPREI
- **ForeCAT** (Provides the coronal trajectory outputs (final latitude, longitude, tilt) that serve as correlated outputs in the table)
- **ANTEATR** (Provides interplanetary propagation outputs (arrival time, speed, size, shape) used as correlated outputs)
- **FIDO** (Provides synthetic in situ magnetic field and velocity profiles used as correlated outputs)

## Examples

```
python processOSPREI.py OSPtestcase.txt
```

## Evaluation signals

- All tabulated correlation coefficients lie within [-1, 1]
- Each per-CME table reproduces the reported structure of 6 varied inputs and 11 outputs exceeding the 0.5 cutoff
- CME 1 exhibits strong correlations of final latitude, longitude, and tilt with their initial values
- The set of retained pairs is drawn from the varied-parameter ranges defined in the '.ens' ensemble configuration files
- Tables differ between the two CMEs (not entirely the same parameters exceed the cutoff for each), consistent with case-specific behavior

## Limitations

- Only two CME cases (22 April and 09 May 2021) were considered; more cases are needed to establish the model
- Correlation tables are case-specific — the same inputs and outputs do not exceed the cutoff for both CMEs
- Ensemble correlation plots in processOSPREI.py require significant computation and may need to be commented out for quick figure generation during testing

## Evidence

- [readme] Ensemble table loading via config file: "If it is set greater than 1 then the program looks for a second file with the same name as the input file but extension ".ens" that sets the parameters varied in the ensemble and their range."
- [readme] Results object and figure generation: "This takes the standardized output files, processes them into a results object, then makes an assortment of figures depending on whether a single simulation or a full ensemble was run."
- [intro] Ensemble purpose in article: "OSPREI includes ensemble modeling by varying each input parameter to probe any uncertainty in their values, yielding probabilities for all outputs."
- [results] Correlation cutoff from task card: [evidence span withheld — anchor status: not_found]
