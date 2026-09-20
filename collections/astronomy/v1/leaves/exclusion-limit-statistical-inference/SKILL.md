---
name: exclusion-limit-statistical-inference
description: Use when the goal is to convert null-result ionization-rate data from
  argon or xenon target DM searches, plus tabulated atomic response functions, into
  coupling-vs-mass exclusion limit curves for DM-electron interaction operators.
when_to_use_negative:
- Input is experimental data with a detected signal rather than a null result — the
  limit-setting procedure equates predicted rate to null-result sensitivity and does
  not apply
- The atomic response functions have not yet been computed/tabulated for the target
  atom (e.g. targets other than argon or xenon); run the DarkARC/DarkART response
  tabulation first
- The analysis concerns DM-nucleus scattering rather than DM-electron interactions;
  the four atomic responses here describe atomic ionization via DM-electron scattering
edam_operation: http://edamontology.org/operation_3659
tools:
- name: DarkARC
  role: Python tool that computes and tabulates the atomic response functions (from
    three radial integrals) consumed as inputs when setting the exclusion limits
  repo: https://github.com/temken/DarkARC
- name: DarkART
  role: C++ successor of DarkARC that computes the same four atomic response functions;
    use it when the deprecated Python tool is unavailable
  repo: https://github.com/temken/DarkART
- name: Python
  role: Language used for DarkARC's response tabulation (radial_integrals_tabulation.py,
    atomic_responses_tabulation.py) and for the limit-scanning workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1103/physrevresearch.2.033195
    title: Atomic responses to general dark matter-electron interactions
  source_content_hash: sha256:29fd374c0b3944ca14966dd63bc521a8fd69468b165054d7be0aca5e65854018
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/temken/DarkARC
  tools:
  - DarkARC
  - DarkART
  - Python
  edam_operation: http://edamontology.org/operation_3659
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# exclusion-limit-statistical-inference

## Summary

Derives 90% confidence level exclusion limits on the strength of DM-electron interaction operators by scanning the coupling until the predicted atomic ionization rate (built from tabulated atomic responses for argon and xenon) equals the null-result sensitivity of direct detection experiments.

## When to use

Use when the goal is to convert null-result ionization-rate data from argon or xenon target DM searches, plus tabulated atomic response functions, into coupling-vs-mass exclusion limit curves for DM-electron interaction operators.

## When NOT to use

- Input is experimental data with a detected signal rather than a null result — the limit-setting procedure equates predicted rate to null-result sensitivity and does not apply
- The atomic response functions have not yet been computed/tabulated for the target atom (e.g. targets other than argon or xenon); run the DarkARC/DarkART response tabulation first
- The analysis concerns DM-nucleus scattering rather than DM-electron interactions; the four atomic responses here describe atomic ionization via DM-electron scattering

## Inputs

- Tabulated atomic response function tables for argon and xenon (four independent responses) from the DarkARC /data output folder
- Null-result ionization-rate data of argon and xenon target DM search experiments (as referenced in arXiv:1912.08204 / Zenodo DOI:10.5281/zenodo.3581334)
- List of DM-electron interaction operators and dark matter mass points to scan

## Outputs

- exclusion_limits.csv — exclusion limit curves (coupling strength vs. DM mass) per operator and target (argon, xenon) at 90% C.L.

## How to apply

Load the tabulated atomic response function tables for argon and xenon from the /data output folder of the DarkARC pipeline, where responses are built from three tabulated radial integrals combined via atomic_responses_tabulation.py. For each DM-electron interaction operator and dark matter mass, compute the predicted ionization rate as the operator-dependent combination of the four independent atomic responses identified in arXiv:1912.08204. Load the null-result ionization-rate data of the argon and xenon target experiments referenced in arXiv:1912.08204 and its supplementary deposit (DOI:10.5281/zenodo.3581334). Scan the interaction coupling strength and, per operator and mass point, determine the coupling at which the predicted rate equals the experimental null-result sensitivity under the 90% confidence level criterion. Assemble the resulting coupling-vs-DM-mass exclusion limit curves for both targets and validate them against the published limits in arXiv:1912.08204.

## Related tools

- **DarkARC** (Python tool that computes and tabulates the atomic response functions (from three radial integrals) consumed as inputs when setting the exclusion limits) — https://github.com/temken/DarkARC
- **DarkART** (C++ successor of DarkARC that computes the same four atomic response functions; use it when the deprecated Python tool is unavailable) — https://github.com/temken/DarkART
- **Python** (Language used for DarkARC's response tabulation (radial_integrals_tabulation.py, atomic_responses_tabulation.py) and for the limit-scanning workflow)

## Examples

```
python src/atomic_responses_tabulation.py  # combine tabulated radial integrals into response tables, then scan couplings against null-result rates to build exclusion_limits.csv
```

## Evaluation signals

- Reproduced 90% C.L. limit curves match the published exclusion limits in arXiv:1912.08204 for the corresponding operators and targets (argon, xenon)
- For every operator/mass point, the reported coupling is the one where predicted ionization rate equals the experimental null-result sensitivity (rate equality check)
- The predicted rate reconstructs as the correct operator-dependent combination of the four independent atomic responses
- exclusion_limits.csv contains coupling-vs-DM-mass curves for both argon and xenon targets covering the scanned operator set
- Atomic response tables trace back to the three tabulated radial integrals produced by radial_integrals_tabulation.py

## Limitations

- DarkARC (Python) is no longer maintained and was replaced by the C++ tool DarkART; new work should prefer DarkART (openmp-based parallelism instead of multiprocessing)
- The method is specific to sub-GeV dark matter direct detection and to targets (argon, xenon) for which atomic response functions have been tabulated
- Limits are statistical statements at a fixed confidence level (90% C.L.) and are only valid for the interaction operators covered by the four-response formalism

## Evidence

- [abstract] 90% C.L. exclusion limits from null results: [evidence span withheld — anchor status: not_found]
- [methods] coupling scan procedure: [evidence span withheld — anchor status: not_found]
- [abstract] four atomic responses: "the rate at which atoms can be ionized via DM-electron scattering can in general be expressed in terms of four independent atomic responses, three of which we identify here for the first time"
- [readme] response tabulation pipeline: "the computation and tabulation of three radial integrals (via */src/radial_integrals_tabulation.py*)."
- [results] validation against published limits: [evidence span withheld — anchor status: not_found]
