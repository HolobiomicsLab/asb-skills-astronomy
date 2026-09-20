---
name: tabular-result-aggregation-and-export
description: Use when multiple response-function tables from /data (DarkARC output)
  must be reduced to one comparative summary, e.g. ratios of new vs. known atomic
  responses per DM-electron operator and target.
when_to_use_negative:
- The raw radial integral tables (src/radial_integrals_tabulation.py output) have
  not yet been combined into atomic response function tables
- The task requires recomputing response functions from wave function overlap integrals
  rather than aggregating existing tabulated values
- Input tables come from DarkART rather than DarkARC, since DarkARC output conventions
  may not match and DarkARC is no longer maintained
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: DarkARC
  role: Produces the tabulated atomic response function tables in /data that serve
    as the aggregation input
  repo: https://github.com/temken/DarkARC
- name: Python
  role: Runtime environment for the pandas/numpy ratio computation and CSV export
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1103/physrevresearch.2.033195
    title: Atomic responses to general dark matter-electron interactions
  source_content_hash: sha256:32e8b9d4fe0e95fe528209bc6b37a2076c26c2e45ea997f6492d2fd2db0c25d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/temken/DarkARC
  tools:
  - DarkARC
  - Python
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# tabular-result-aggregation-and-export

## Summary

Aggregates per-case numerical ratios from DarkARC-generated atomic response function tables into a single pandas summary table and exports it as a CSV, quantifying how numerically important the three newly identified atomic responses are relative to the previously known ionization form factor.

## When to use

Use when multiple response-function tables from /data (DarkARC output) must be reduced to one comparative summary, e.g. ratios of new vs. known atomic responses per DM-electron operator and target.

## When NOT to use

- The raw radial integral tables (src/radial_integrals_tabulation.py output) have not yet been combined into atomic response function tables
- The task requires recomputing response functions from wave function overlap integrals rather than aggregating existing tabulated values
- Input tables come from DarkART rather than DarkARC, since DarkARC output conventions may not match and DarkARC is no longer maintained

## Inputs

- Atomic response function tables for argon and xenon targets from the DarkARC /data output folder
- List of parameter cases (effective-theory DM-electron interaction operators and targets) from arXiv:1912.08204

## Outputs

- new_response_contributions.csv — one row per parameter case with the relative magnitude (ratio) of each of the three new atomic responses

## How to apply

Load the atomic response function tables from the /data output folder that DarkARC produced by combining radial integrals via src/atomic_responses_tabulation.py. Retrieve the parameter cases — the effective-theory DM-electron interaction operators and atomic targets (argon, xenon) — identified in arXiv:1912.08204. For each case, compute with pandas/numpy the ratio of each of the three newly identified responses to the previously known response (the ionization form factor), then aggregate these ratios into a single summary table listing, per case, the relative magnitude of each new response. Write the aggregated table to new_response_contributions.csv and check that every parameter case from the paper is covered and that the ratios support the finding that the new responses are numerically important in a variety of cases.

## Related tools

- **DarkARC** (Produces the tabulated atomic response function tables in /data that serve as the aggregation input) — https://github.com/temken/DarkARC
- **Python** (Runtime environment for the pandas/numpy ratio computation and CSV export)

## Examples

```
import pandas as pd, numpy as np; df = pd.read_csv('data/atomic_responses.csv'); ratios = df[['W1','W2','W3']].div(df['W0'], axis=0); ratios.to_csv('new_response_contributions.csv')
```

## Evaluation signals

- The CSV contains a row for every parameter case (operator × target) identified in arXiv:1912.08204
- Ratios are computed as new response / previously known response (ionization form factor) and are finite, positive numbers
- Cases with large ratios collectively reproduce the paper's finding that the new responses are numerically important in a variety of cases
- The exported file is valid CSV loadable by pandas.read_csv with one column per new response
- No parameter case from the paper is missing or duplicated in the summary table

## Limitations

- DarkARC is no longer maintained and was replaced by the C++ tool DarkART, so its output conventions may need adaptation
- Ratios depend on the specific operator-target cases tabulated; interpolating to untabulated cases is not supported by this skill
- The conclusion of numerical importance holds for the argon and xenon targets and effective-theory operators investigated in the paper

## Evidence

- [abstract] New responses numerically important: "We find our new atomic responses to be numerically important in a variety of cases, which we identify and investigate thoroughly using effective theory methods"
- [abstract] Four independent responses: "the rate at which atoms can be ionized via DM-electron scattering can in general be expressed in terms of four independent atomic responses, three of which we identify here for the first time"
- [methods] Response tabulation pipeline: "their combination into the response function tables (via */src/atomic_responses_tabulation.py*)"
- [readme] Output folder location: "- */data/*: Destination folder of the code's output (tables of integration methods, radial integrals, and finally atomic response functions)."
