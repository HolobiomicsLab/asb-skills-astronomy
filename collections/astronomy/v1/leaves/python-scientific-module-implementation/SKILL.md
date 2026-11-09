---
name: python-scientific-module-implementation
description: Use when the goal is to reconstruct or reimplement, as a runnable Python
  3.10 module, how SMART assembles its model space from the four radiative transfer
  library types for ultraviolet-to-millimetre galaxy SED fitting.
when_to_use_negative:
- The task is to run the full SMART Bayesian MCMC fit on observational data (e.g.
  Spitzer spectroscopy of HERUS ULIRGs) rather than to reimplement how its model space
  is assembled
- The input is photometric or spectroscopic galaxy data needing an SED fit, not the
  pre-computed radiative transfer library templates themselves
- A non-radiative-transfer model space (e.g. popular energy balance SED methods) is
  what must be reproduced
edam_operation: http://edamontology.org/operation_0244
tools:
- name: SMART
  role: Source open-source SED fitting code whose four-library model space and component-combination
    logic the module reproduces
  repo: https://github.com/ch-var/SMART
- name: Python
  role: Implementation language (Python 3.10, Anaconda package) in which the component-combination
    module is developed and executed
  repo: https://github.com/ch-var/SMART.git
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/mnras/stae1141
    title: 'SMART: spectral energy distributions Markov chain analysis with radiative
      transfer models'
  source_content_hash: sha256:d9fe367e834c1031bb08ee27ced8ec615340ffba32bbe2895eb8d50b75adb049
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ch-var/SMART
  tools:
  - Python
  - SMART
  edam_operation: http://edamontology.org/operation_0244
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# python-scientific-module-implementation

## Summary

Implement a Python 3.10 scientific module that reproduces a component of the SMART SED-fitting pipeline: loading the four pre-computed radiative transfer libraries (starburst, AGN torus, host galaxy, polar dust) and combining scaled templates into a composite ultraviolet-to-millimetre galaxy SED.

## When to use

Use when the goal is to reconstruct or reimplement, as a runnable Python 3.10 module, how SMART assembles its model space from the four radiative transfer library types for ultraviolet-to-millimetre galaxy SED fitting.

## When NOT to use

- The task is to run the full SMART Bayesian MCMC fit on observational data (e.g. Spitzer spectroscopy of HERUS ULIRGs) rather than to reimplement how its model space is assembled
- The input is photometric or spectroscopic galaxy data needing an SED fit, not the pre-computed radiative transfer library templates themselves
- A non-radiative-transfer model space (e.g. popular energy balance SED methods) is what must be reproduced

## Inputs

- SMART_v1.0.7z release archive containing the pre-computed radiative transfer model libraries
- Four library types: starburst, AGN torus, host galaxy, and polar dust
- SMART_User_Manual.pdf for library file formats and usage
- User-specified selection of library components to combine

## Outputs

- Composite multi-component ultraviolet-to-millimetre SED assembled exclusively from the four radiative transfer library types
- component_combination.py module executable end-to-end in Python 3.10

## How to apply

Extract the SMART v1.0 release archive (SMART_v1.0.7z, or SMART_v1.0_mac.7z on macOS) from the ch-var/SMART repository and load the four pre-computed radiative transfer library types, following the file-format guidance in SMART_User_Manual.pdf. Implement the component-combination function so that each user-selected library template (starburst, AGN torus, host galaxy, polar dust) is scaled and the components are summed into a single composite SED, mirroring the model space SMART explores with its Bayesian Markov chain Monte Carlo method. The decision point is the user-specified component selection, which determines which of the four library types enter the composite. Validate by executing the module end-to-end and confirming the generated composite SED is assembled exclusively from the four radiative transfer library types, checked via script_runs and file_exists on component_combination.py and its output.

## Related tools

- **SMART** (Source open-source SED fitting code whose four-library model space and component-combination logic the module reproduces) — https://github.com/ch-var/SMART
- **Python** (Implementation language (Python 3.10, Anaconda package) in which the component-combination module is developed and executed) — https://github.com/ch-var/SMART.git

## Examples

```
python component_combination.py  # load the four SMART library types, scale and sum selected templates to output a composite UV-to-mm SED
```

## Evaluation signals

- script_runs: component_combination.py executes end-to-end in Python 3.10 without error
- file_exists: the generated composite SED file is produced by the module
- The composite output SED is assembled exclusively from the four radiative transfer library types (starburst, AGN torus, host galaxy, polar dust)
- Scaling and summation of the selected templates reproduces a single ultraviolet-to-millimetre SED, mirroring the model space SMART fits via Bayesian MCMC

## Limitations

- SMART's model space currently contains only four pre-computed library types, so the composite SED is limited to starburst, AGN torus, host galaxy, and polar dust components
- Library file formats must be loaded per SMART_User_Manual.pdf; platform-specific archives exist (SMART_v1.0.7z for Windows/Linux, SMART_v1.0_mac.7z for macOS)
- No changelog was found for the repository, so version drift in the release archive contents cannot be tracked from metadata alone

## Evidence

- [abstract] SMART fits the ultraviolet to millimetre SEDs of galaxies exclusively with radiative transfer models that currently constitute four types of pre-computed libraries: "SMART fits the ultraviolet to millimetre SEDs of galaxies exclusively with radiative transfer models that currently constitute four types of pre-computed libraries"
- [readme] which describe the starburst, AGN torus, host galaxy and polar dust components: "which describe the starburst, active galactic nucleus torus, host galaxy and polar dust components"
- [intro] The code is developed in PYTHON 3.10 (Anaconda package).: [evidence span withheld — anchor status: not_found]
