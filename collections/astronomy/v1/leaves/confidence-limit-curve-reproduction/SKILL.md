---
name: confidence-limit-curve-reproduction
description: Use when null-result direct detection data (e.g. argon/xenon ionization
  searches) must be converted into per-operator, per-mass exclusion limit curves on
  DM-electron coupling strength, given pre-computed atomic response tables.
when_to_use_negative:
- The required atomic response tables have not yet been computed — run the DarkARC
  radial_integrals_tabulation.py and atomic_responses_tabulation.py steps first
- The task is computing/tabulating atomic responses themselves rather than converting
  existing responses into exclusion limits (that is the atomic_responses_tabulation
  skill)
- Experiments with observed (non-null) signals, where a limit-setting scan against
  a null-result sensitivity does not apply
edam_operation: http://edamontology.org/operation_2424
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: DarkARC
  role: Source of the tabulated atomic response function tables in /data that are
    combined into predicted ionization rates per operator
  repo: https://github.com/temken/DarkARC
- name: DarkART
  role: C++ successor to DarkARC for computing the four atomic response functions
    if re-computation of responses is needed
  repo: https://github.com/temken/DarkART
- name: Python
  role: Implementation language for scanning coupling strengths, assembling limit
    curves, and writing exclusion_limits.csv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1103/physrevresearch.2.033195
    title: Atomic responses to general dark matter-electron interactions
  source_content_hash: sha256:4e3e5960650e6affe6da41167e436a612b6baadee5c2b2c8de2e47014185df52
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/temken/DarkARC
  tools:
  - DarkARC
  - DarkART
  - Python
  edam_operation: http://edamontology.org/operation_2424
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# confidence-limit-curve-reproduction

## Summary

Reproduces 90% confidence level exclusion limits on the strength of DM-electron interactions by scanning coupling strengths against null-result ionization rates from argon and xenon target experiments, using the four tabulated atomic response functions from arXiv:1912.08204 (DarkARC pipeline).

## When to use

Use when null-result direct detection data (e.g. argon/xenon ionization searches) must be converted into per-operator, per-mass exclusion limit curves on DM-electron coupling strength, given pre-computed atomic response tables.

## When NOT to use

- The required atomic response tables have not yet been computed — run the DarkARC radial_integrals_tabulation.py and atomic_responses_tabulation.py steps first
- The task is computing/tabulating atomic responses themselves rather than converting existing responses into exclusion limits (that is the atomic_responses_tabulation skill)
- Experiments with observed (non-null) signals, where a limit-setting scan against a null-result sensitivity does not apply

## Inputs

- Tabulated atomic response function tables for argon and xenon from the DarkARC /data output folder (built from the three tabulated radial integrals)
- Null-result ionization-rate data from argon and xenon target DM search experiments (arXiv:1912.08204 supplementary data, DOI:10.5281/zenodo.3581334)
- List of DM-electron interaction operators and DM mass grid points

## Outputs

- exclusion_limits.csv containing 90% C.L. exclusion limit curves (coupling strength vs. DM mass) for each operator and for both argon and xenon targets

## How to apply

Load the tabulated atomic response function tables for argon and xenon from the /data output folder of the DarkARC pipeline, where each response is the operator-dependent combination of the four independent atomic responses built from three tabulated radial integrals. Load the null-result ionization-rate data of the argon and xenon target experiments referenced in arXiv:1912.08204 (supplementary data DOI:10.5281/zenodo.3581334). For each DM-electron interaction operator and each DM mass, compute the predicted ionization rate as the operator-dependent combination of the four independent atomic responses. Scan the interaction coupling strength and, per operator and mass point, find the coupling at which the predicted ionization rate equals the experimental null-result sensitivity under the 90% C.L. criterion. Assemble coupling-strength-vs-DM-mass limit curves for both targets and save them as exclusion_limits.csv, then validate against the published limits.

## Related tools

- **DarkARC** (Source of the tabulated atomic response function tables in /data that are combined into predicted ionization rates per operator) — https://github.com/temken/DarkARC
- **DarkART** (C++ successor to DarkARC for computing the four atomic response functions if re-computation of responses is needed) — https://github.com/temken/DarkART
- **Python** (Implementation language for scanning coupling strengths, assembling limit curves, and writing exclusion_limits.csv)

## Examples

```
python scripts/set_limits.py --target argon xenon --responses /data --sensitivity data/null_results --cl 0.9 --output exclusion_limits.csv
```

## Evaluation signals

- Reproduced limit curves match the published 90% C.L. exclusion limits reported in arXiv:1912.08204 for the corresponding operators and targets
- Each operator/mass-point limit corresponds to the coupling where the predicted ionization rate exactly equals the experimental null-result sensitivity
- exclusion_limits.csv exists, is parseable, and covers all requested operators, mass points, and both targets (argon, xenon)
- Predicted ionization rates use the correct operator-dependent combination of the four independent atomic responses

## Limitations

- DarkARC is no longer maintained and was replaced by the C++ tool DarkART; users needing maintained tooling should use DarkART
- Only limits at the 90% confidence level are covered; other C.L. criteria are not implemented
- Applicable only to argon and xenon targets and the DM-electron operators studied in arXiv:1912.08204

## Evidence

- [abstract] 90% C.L. exclusion limits from null results: [evidence span withheld — anchor status: not_found]
- [abstract] Rate expressed via four independent atomic responses: "the rate at which atoms can be ionized via DM-electron scattering can in general be expressed in terms of four independent atomic responses"
- [readme] DarkARC output location and radial integral basis: "/data/: Destination folder of the code's output (tables of integration methods, radial integrals, and finally atomic response functions)."
- [readme] DarkARC deprecated in favor of DarkART: "THE PYTHON TOOL DarkARC IS NO LONGER MAINTAINED AND WAS REPLACED BY THE C++ TOOL"
- [methods] Supplementary data archive for experimental rates: "10.5281/zenodo.3581334"
