---
name: radiative-transfer-library-combination
description: Use when building or inspecting the composite model SEDs SMART fits —
  e.g. reproducing how starburst, AGN torus, host galaxy, and polar dust library templates
  are combined into a single UV-to-mm SED before Bayesian MCMC fitting.
when_to_use_negative:
- The goal is SED fitting with empirical/energy-balance methods rather than radiative
  transfer model libraries — SMART fits SEDs exclusively with radiative transfer models
- Only a single-component SED is needed with no combination of starburst, AGN torus,
  host galaxy, or polar dust libraries
- The SED wavelength coverage does not span the ultraviolet-to-millimetre range the
  SMART libraries are built for
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: SMART
  role: Provides the four pre-computed radiative transfer model libraries (starburst,
    AGN torus, host galaxy, polar dust) that are scaled and summed into composite
    SEDs
  repo: https://github.com/ch-var/SMART.git
- name: Python 3.10
  role: Language (Anaconda package) in which the libraries are loaded and the component-combination
    function is implemented and executed
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/mnras/stae1141
    title: 'SMART: spectral energy distributions Markov chain analysis with radiative
      transfer models'
  source_content_hash: sha256:cd7760dbe2c8ebddccc906e855b86ea16a79ac5ff8377aaab1ed6cdc46630978
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ch-var/SMART.git
  tools:
  - Python 3.10
  - SMART
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# radiative-transfer-library-combination

## Summary

Assemble composite ultraviolet-to-millimetre galaxy SEDs by scaling and summing templates from SMART's four pre-computed radiative transfer libraries (starburst, AGN torus, host galaxy, polar dust), mirroring the model space SMART fits with Bayesian MCMC.

## When to use

Use when building or inspecting the composite model SEDs SMART fits — e.g. reproducing how starburst, AGN torus, host galaxy, and polar dust library templates are combined into a single UV-to-mm SED before Bayesian MCMC fitting.

## When NOT to use

- The goal is SED fitting with empirical/energy-balance methods rather than radiative transfer model libraries — SMART fits SEDs exclusively with radiative transfer models
- Only a single-component SED is needed with no combination of starburst, AGN torus, host galaxy, or polar dust libraries
- The SED wavelength coverage does not span the ultraviolet-to-millimetre range the SMART libraries are built for

## Inputs

- SMART v1.0 release archive (SMART_v1.0.7z or SMART_v1.0_mac.7z) containing the pre-computed radiative transfer model libraries
- User-specified selections of library components (starburst, AGN torus, host galaxy, polar dust) and their scaling parameters
- SMART_User_Manual.pdf guidance on library file formats

## Outputs

- Composite ultraviolet-to-millimetre SED assembled from the selected radiative transfer library components (e.g. via component_combination.py)

## How to apply

Obtain and extract the SMART v1.0 release archive (SMART_v1.0.7z for Windows/Linux, SMART_v1.0_mac.7z for macOS), which contains the pre-computed radiative transfer model libraries. Load the four library types — starburst, AGN torus, host galaxy, and polar dust — in Python 3.10 (Anaconda package), following the library-format guidance in SMART_User_Manual.pdf. Implement the component-combination step: scale each user-selected library template and sum the components to produce a composite ultraviolet-to-millimetre SED, mirroring the model space SMART explores with its Bayesian Markov chain Monte Carlo method. Validate by running the combination end-to-end and checking that the output SED is assembled exclusively from the four radiative transfer library types.

## Related tools

- **SMART** (Provides the four pre-computed radiative transfer model libraries (starburst, AGN torus, host galaxy, polar dust) that are scaled and summed into composite SEDs) — https://github.com/ch-var/SMART.git
- **Python 3.10** (Language (Anaconda package) in which the libraries are loaded and the component-combination function is implemented and executed)

## Examples

```
python component_combination.py --components starburst agn_torus host_galaxy polar_dust --output composite_sed.dat
```

## Evaluation signals

- The component-combination module runs end-to-end without error (script_runs on component_combination.py)
- The generated composite SED file exists on disk (file_exists check)
- The composite output is a SED assembled exclusively from the four radiative transfer library types (starburst, AGN torus, host galaxy, polar dust)
- The composite SED covers the ultraviolet-to-millimetre wavelength range, consistent with SMART's fitting domain

## Limitations

- SMART's model space is limited to the four pre-computed library types; no other radiative transfer or empirical templates can be combined
- The code is developed in Python 3.10 (Anaconda package); other environments may not be supported
- No changelog is available for the repository, so version drift in library formats may be hard to track

## Evidence

- [abstract] SMART fits the ultraviolet to millimetre SEDs of galaxies exclusively with radiative transfer models that currently constitute four types of pre-computed libraries: "SMART fits the ultraviolet to millimetre SEDs of galaxies exclusively with radiative transfer models that currently constitute four types of pre-computed libraries"
- [intro] The code is developed in PYTHON 3.10 (Anaconda package).: [evidence span withheld — anchor status: not_found]
- [intro] Implementing a Bayesian Markov chain Monte Carlo method, SMART fits the ultraviolet to millimetre SEDs of galaxies exclusively with radiative transfer models: [evidence span withheld — anchor status: not_found]
- [readme] SMART is developed in Python and is available at https://github.com/ch-var/SMART.git: "SMART is developed in Python and is available at https://github.com/ch-var/SMART.git"
- [readme] To learn how to run SMART and for a helpful user guide, please read the manual.: "To learn how to run <b>SMART</b> and for a helpful user guide, please read the <a href="https://github.com/ch-var/SMART/blob/main/assets/SMART_User_Manual.pdf">manual</a>."
