---
name: dm-electron-operator-mapping
description: Use when null-result ionization data from argon or xenon DM search experiments
  must be converted into 90% C.L. exclusion limits on effective DM-electron operator
  couplings as a function of DM mass.
when_to_use_negative:
- The atomic response tables for the target element have not yet been computed — first
  run radial_integrals_tabulation.py and atomic_responses_tabulation.py
- The experiment reported a signal or non-ionization-based observable rather than
  a null ionization-rate result, since the limit criterion assumes null-result sensitivity
- The task is to compute/tabulate atomic responses themselves rather than map them
  to experimental limits (that is a separate upstream skill)
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3452
tools:
- name: DarkARC
  role: Python tool providing the tabulated atomic response function tables for argon
    and xenon used as inputs to the limit computation
  repo: https://github.com/temken/DarkARC
- name: DarkART
  role: C++ successor tool that computes the four atomic response functions introduced
    in arXiv:1912.08204 (DarkARC is deprecated)
  repo: https://github.com/temken/DarkART
- name: Python
  role: Used to load response tables, compute operator-dependent rate combinations,
    scan couplings, and write exclusion_limits.csv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1103/physrevresearch.2.033195
    title: Atomic responses to general dark matter-electron interactions
  source_content_hash: sha256:d94ea051aa894cdfecf058aaafd83f9cb0932e53d8f0a97f17e62ed04994d837
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/temken/DarkARC
  tools:
  - DarkARC
  - DarkART
  - Python
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# dm-electron-operator-mapping

## Summary

Derives 90% confidence level exclusion limits on the strength of DM-electron interaction operators by mapping tabulated atomic response functions (argon and xenon) onto experimental null-result ionization rates, producing coupling-vs-DM-mass limit curves.

## When to use

Use when null-result ionization data from argon or xenon DM search experiments must be converted into 90% C.L. exclusion limits on effective DM-electron operator couplings as a function of DM mass.

## When NOT to use

- The atomic response tables for the target element have not yet been computed — first run radial_integrals_tabulation.py and atomic_responses_tabulation.py
- The experiment reported a signal or non-ionization-based observable rather than a null ionization-rate result, since the limit criterion assumes null-result sensitivity
- The task is to compute/tabulate atomic responses themselves rather than map them to experimental limits (that is a separate upstream skill)

## Inputs

- Tabulated atomic response function tables for argon and xenon (DarkARC /data output, built from three tabulated radial integrals)
- Null-result ionization-rate data from argon and xenon target DM search experiments (arXiv:1912.08204 supplementary data, DOI:10.5281/zenodo.3581334)
- List of DM-electron interaction operators and DM mass grid points

## Outputs

- exclusion_limits.csv containing 90% C.L. exclusion limit curves (coupling strength vs. DM mass) per operator for argon and xenon targets

## How to apply

Load the tabulated atomic response function tables for argon and xenon from the /data output folder of the DarkARC pipeline, where responses are built from three tabulated radial integrals (src/radial_integrals_tabulation.py combined via src/atomic_responses_tabulation.py). Load the null-result ionization-rate data of the argon and xenon target experiments referenced in arXiv:1912.08204 and its deposited supplementary data (DOI:10.5281/zenodo.3581334). For each DM-electron interaction operator and each DM mass, compute the predicted ionization rate as the operator-dependent combination of the four independent atomic responses. Scan the coupling strength and find, per operator and mass point, the coupling at which the predicted rate equals the experimental sensitivity under the 90% C.L. criterion. Assemble the coupling-vs-mass exclusion curves for both targets and save as exclusion_limits.csv, then validate against the published limits in arXiv:1912.08204.

## Related tools

- **DarkARC** (Python tool providing the tabulated atomic response function tables for argon and xenon used as inputs to the limit computation) — https://github.com/temken/DarkARC
- **DarkART** (C++ successor tool that computes the four atomic response functions introduced in arXiv:1912.08204 (DarkARC is deprecated)) — https://github.com/temken/DarkART
- **Python** (Used to load response tables, compute operator-dependent rate combinations, scan couplings, and write exclusion_limits.csv)

## Examples

```
python src/atomic_responses_tabulation.py  # then load /data response tables to scan couplings per operator and mass, writing exclusion_limits.csv
```

## Evaluation signals

- Reproduced limit curves match the published 90% C.L. exclusion limits in arXiv:1912.08204 for the corresponding operators and targets
- For each operator and mass point, the reported coupling is the one where predicted ionization rate equals the experimental null-result sensitivity (90% C.L. criterion)
- Curves exist for both argon and xenon targets and cover the wide range of DM-electron interaction operators
- exclusion_limits.csv is well-formed with coupling strength vs. DM mass per operator/target

## Limitations

- DarkARC is no longer maintained and was replaced by the C++ tool DarkART, so new work should prefer DarkART
- Limits are valid only for sub-GeV DM-electron scattering with argon and xenon atomic targets covered by the tabulated responses
- Only three of the four independent atomic responses were newly identified in this work; the ionization form factor corresponds to the first response

## Evidence

- [abstract] 90% C.L. exclusion limits derived from null results using operator-dependent combinations of four atomic responses: [evidence span withheld — anchor status: not_found]
- [abstract] Ionization rate expressible in terms of four independent atomic responses, three identified for the first time: "the rate at which atoms can be ionized via DM-electron scattering can in general be expressed in terms of four independent atomic responses, three of which we identify here for the first time"
- [methods] Tabulation of three radial integrals and their combination into atomic response function tables: "the computation and tabulation of three radial integrals (via */src/radial_integrals_tabulation.py*)"
- [readme] DarkARC replaced by DarkART, which computes the four response functions in parallel with OpenMP: [evidence span withheld — anchor status: external_doc]
