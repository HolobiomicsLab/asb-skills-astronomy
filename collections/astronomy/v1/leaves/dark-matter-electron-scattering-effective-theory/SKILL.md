---
name: dark-matter-electron-scattering-effective-theory
description: Use when modeling sub-GeV dark matter-electron ionization of atomic targets
  (argon, xenon) under general effective-theory interaction operators, and you need
  the tabulated atomic responses — not just the ionization form factor — or must assess
  the relative weight of the three newly identified.
when_to_use_negative:
- The analysis concerns DM-nucleus (nuclear recoil) scattering rather than DM-electron
  ionization of atomic targets
- Only the standard ionization form factor is required for a single, already-known
  DM-electron interaction — the new-response decomposition is unnecessary overhead
- 'You intend to run DarkARC for new production computations: the Python tool is no
  longer maintained and was replaced by the C++ tool DarkART'
edam_operation: http://edamontology.org/operation_0306
tools:
- name: DarkARC
  role: Python tool that computes and tabulates the four atomic response functions
    for argon and xenon; combines three radial integrals into response tables via
    atomic_responses_tabulation.py in parallel with multiprocessing.
  repo: https://github.com/temken/DarkARC
- name: DarkART
  role: C++ successor to DarkARC that computes the same four atomic response functions
    with OpenMP parallelization; preferred for new production runs.
  repo: https://github.com/temken/DarkART
- name: Python
  role: Runtime for DarkARC's radial-integral and response tabulation scripts (multiprocessing-parallelized)
    and for post-hoc per-case ratio analysis of new vs. known responses with pandas/numpy.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1103/physrevresearch.2.033195
    title: Atomic responses to general dark matter-electron interactions
  source_content_hash: sha256:5b6f731f59a8e82f4cc7cc87eb105041c5da7e8d900e02e298740f1e67439c64
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/temken/DarkARC
  tools:
  - DarkARC
  - DarkART
  - Python
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# dark-matter-electron-scattering-effective-theory

## Summary

Skill for decomposing the DM-electron ionization rate in atoms into four independent atomic response functions (three newly identified in Catena et al. 2020, PRR 2, 033195) and quantifying their numerical importance across effective-theory operator cases, using DarkARC/DarkART response tables for argon and xenon targets.

## When to use

Use when modeling sub-GeV dark matter-electron ionization of atomic targets (argon, xenon) under general effective-theory interaction operators, and you need the tabulated atomic responses — not just the ionization form factor — or must assess the relative weight of the three newly identified responses.

## When NOT to use

- The analysis concerns DM-nucleus (nuclear recoil) scattering rather than DM-electron ionization of atomic targets
- Only the standard ionization form factor is required for a single, already-known DM-electron interaction — the new-response decomposition is unnecessary overhead
- You intend to run DarkARC for new production computations: the Python tool is no longer maintained and was replaced by the C++ tool DarkART

## Inputs

- Tabulated radial integrals produced by src/radial_integrals_tabulation.py (output in the /data folder)
- Atomic response function tables from src/atomic_responses_tabulation.py for argon and xenon targets
- Effective-theory DM-electron interaction operator parameter cases from arXiv:1912.08204

## Outputs

- Four tabulated atomic response functions per target, including the ionization form factor and the three newly identified responses
- Summary table (e.g. new_response_contributions.csv) listing, per parameter case, the relative magnitude of each new response versus the previously known response
- 90% C.L. exclusion limits on DM-electron interaction strengths when combined with null results of argon/xenon DM search experiments

## How to apply

Express the DM-electron ionization rate as four independent atomic responses, where the first response is the known ionization form factor and the other three are the newly identified responses of arXiv:1912.08204. Generate inputs by tabulating three radial integrals with src/radial_integrals_tabulation.py, then combining them into response function tables with src/atomic_responses_tabulation.py (DarkARC) or the corresponding DarkART C++ pipeline, running in parallel (multiprocessing in DarkARC, OpenMP in DarkART). For each effective-theory operator case from the paper, compute with Python (pandas/numpy) the ratio of each new response to the previously known response for argon and xenon. Aggregate per-case ratios into one summary table, e.g. new_response_contributions.csv. Decision point: if only the standard ionization form factor is needed for a well-known interaction, skip the new-response decomposition; DarkARC is unmaintained, so prefer DarkART for new computations.

## Related tools

- **DarkARC** (Python tool that computes and tabulates the four atomic response functions for argon and xenon; combines three radial integrals into response tables via atomic_responses_tabulation.py in parallel with multiprocessing.) — https://github.com/temken/DarkARC
- **DarkART** (C++ successor to DarkARC that computes the same four atomic response functions with OpenMP parallelization; preferred for new production runs.) — https://github.com/temken/DarkART
- **Python** (Runtime for DarkARC's radial-integral and response tabulation scripts (multiprocessing-parallelized) and for post-hoc per-case ratio analysis of new vs. known responses with pandas/numpy.)

## Examples

```
python src/radial_integrals_tabulation.py && python src/atomic_responses_tabulation.py
```

## Evaluation signals

- Response tables contain exactly four independent atomic responses, with the first response matching the known ionization form factor
- The /data output folder contains the three tabulated radial integrals followed by the combined response function tables, in that pipeline order
- The summary CSV covers all effective-theory parameter cases from the paper for both argon and xenon targets
- Per-case ratios of the three new responses to the previously known response reproduce the paper's conclusion that the new responses are numerically important in a variety of cases
- The computed responses support 90% C.L. exclusion limits consistent with those derived from argon and xenon null results in the paper

## Limitations

- DarkARC is deprecated and no longer maintained; DarkART (C++, v0.1.0, Zenodo DOI 10.5281/zenodo.6046225) supersedes it
- Response computations are expensive and rely on parallelization (multiprocessing in DarkARC, OpenMP in DarkART); serial runs will be slow
- Results are tabulated for the atomic targets used in operating DM search experiments (argon, xenon); other targets are out of scope of this pipeline

## Evidence

- [abstract] Four independent responses and three new ones: "the rate at which atoms can be ionized via DM-electron scattering can in general be expressed in terms of four independent atomic responses, three of which we identify here for the first time"
- [abstract] Numerical importance via effective theory: "We find our new atomic responses to be numerically important in a variety of cases, which we identify and investigate thoroughly using effective theory methods"
- [methods] Two-step tabulation pipeline: [evidence span withheld — anchor status: not_found]
- [readme] First response is the ionization form factor: [evidence span withheld — anchor status: external_doc]
- [intro] DarkARC deprecation: "THE PYTHON TOOL DarkARC IS NO LONGER MAINTAINED AND WAS REPLACED BY THE C++ TOOL DarkART"
