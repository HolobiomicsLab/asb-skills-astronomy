---
name: dark-matter-electron-response-computation
description: Use when parameterizing DM-electron ionization rates of argon or xenon
  detector targets in terms of atomic responses, or when reproducing the radial-integral
  and response-function tables of arXiv:1912.08204.
when_to_use_negative:
- The tool only covers atomic argon and xenon targets; other detector materials are
  outside DarkARC/DarkART scope
- If a maintained, faster implementation is required — DarkARC is deprecated in favor
  of the C++ tool DarkART
- For DM-nucleus (not DM-electron) scattering responses, which these atomic response
  functions do not describe
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_0121
tools:
- name: DarkARC
  role: Python tool whose two scripts (src/radial_integrals_tabulation.py, src/atomic_responses_tabulation.py)
    perform the radial-integral and atomic-response tabulation; deprecated
  repo: https://github.com/temken/DarkARC
- name: DarkART
  role: C++ successor tool computing the same four atomic response functions in parallel
    using OpenMP
  repo: https://github.com/temken/DarkART
- name: multiprocessing
  role: Python library used to parallelize grid-point radial integral and response
    computations across CPU cores
  repo: https://docs.python.org/2/library/multiprocessing.html
provenance:
  source_task_ids:
  - task_002
  - task_001
  - task_003
  source_papers:
  - doi: 10.1103/physrevresearch.2.033195
    title: Atomic responses to general dark matter-electron interactions
  source_content_hash: sha256:8d497e508c0ccedda102ecd900e07fd5c1c40a3bea9d570ebc51e9fcf37d5ad1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/temken/DarkARC
  tools:
  - DarkARC
  - DarkART
  - multiprocessing
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# dark-matter-electron-response-computation

## Summary

Computes and tabulates the four independent atomic response functions describing how argon and xenon atoms respond to general dark matter-electron interactions, via a two-step DarkARC pipeline: tabulation of three radial integrals followed by their combination into response function tables.

## When to use

Use when parameterizing DM-electron ionization rates of argon or xenon detector targets in terms of atomic responses, or when reproducing the radial-integral and response-function tables of arXiv:1912.08204.

## When NOT to use

- The tool only covers atomic argon and xenon targets; other detector materials are outside DarkARC/DarkART scope
- If a maintained, faster implementation is required — DarkARC is deprecated in favor of the C++ tool DarkART
- For DM-nucleus (not DM-electron) scattering responses, which these atomic response functions do not describe

## Inputs

- Archived DarkARC v1.0 source code (Zenodo DOI 10.5281/zenodo.3581334) including src/radial_integrals_tabulation.py and src/atomic_responses_tabulation.py
- Bundled atomic-structure input data for argon and xenon electron wave functions
- Tabulation grid of momentum transfer and electron binding energy values

## Outputs

- Tables of integration methods in /data/
- Tabulated radial integrals (three per grid point) in /data/
- Four atomic response function tables for argon and xenon in /data/

## How to apply

Run the two-step tabulation pipeline of the archived DarkARC v1.0 release (Zenodo DOI 10.5281/zenodo.3581334): first execute src/radial_integrals_tabulation.py to numerically evaluate and tabulate the three radial integrals — electron wave function overlap integrals — over a grid of momentum transfer and electron binding energy, distributing grid-point evaluations across CPU cores with the Python multiprocessing library. These tables are written to the /data/ folder as tables of integration methods and radial integrals. Second, run src/atomic_responses_tabulation.py to combine the tabulated radial integrals into the four atomic response function tables (the first being the ionization form factor) for argon and xenon, also in parallel via multiprocessing and written to /data/. Validate by confirming /data contains the integration-method, radial-integral, and all four response function tables, consistent with the archived release. Note DarkARC is no longer maintained and was superseded by the C++ tool DarkART, which uses OpenMP instead of multiprocessing.

## Related tools

- **DarkARC** (Python tool whose two scripts (src/radial_integrals_tabulation.py, src/atomic_responses_tabulation.py) perform the radial-integral and atomic-response tabulation; deprecated) — https://github.com/temken/DarkARC
- **DarkART** (C++ successor tool computing the same four atomic response functions in parallel using OpenMP) — https://github.com/temken/DarkART
- **multiprocessing** (Python library used to parallelize grid-point radial integral and response computations across CPU cores) — https://docs.python.org/2/library/multiprocessing.html

## Examples

```
python src/radial_integrals_tabulation.py && python src/atomic_responses_tabulation.py
```

## Evaluation signals

- The /data/ folder contains three layers of output: tables of integration methods, tables of radial integrals, and finally the four atomic response function tables
- Radial integral and response tables match the archived DarkARC v1.0 release (Zenodo DOI 10.5281/zenodo.3581334) accompanying arXiv:1912.08204
- All four independent atomic responses are tabulated for both argon and xenon targets, with the first response reproducing the known ionization form factor
- Computations ran in parallel via the multiprocessing library without missing grid points over the momentum-transfer / binding-energy grid

## Limitations

- DarkARC is no longer maintained and was replaced by the C++ tool DarkART
- Tabulation is restricted to argon and xenon atomic wave functions used in operating DM search experiments
- DarkART (the successor) has different dependencies (arb, boost, CMake, GSL, libconfig, libphysica, openmp) and a different parallelization scheme (OpenMP vs multiprocessing)

## Evidence

- [readme] two-step tabulation pipeline: "The tabulation of the atomic response functions is separated into two steps: - the computation and tabulation of three radial integrals (via */src/radial_integrals_tabulation.py*)."
- [methods] parallelization: "The computations are performed in parallel using the [*multiprocessing*](https://docs.python.org/2/library/multiprocessing.html) library."
- [abstract] four responses finding: "the rate at which atoms can be ionized via DM-electron scattering can in general be expressed in terms of four independent atomic responses, three of which we identify here for the first time"
- [intro] deprecation note: "THE PYTHON TOOL DarkARC IS NO LONGER MAINTAINED AND WAS REPLACED BY THE C++ TOOL DarkART"
- [readme] output destination: "*/data/*: Destination folder of the code's output (tables of integration methods, radial integrals, and finally atomic response functions)."
