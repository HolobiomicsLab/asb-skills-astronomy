---
name: galaxy-sed-model-space-assembly
description: Use when building a multi-component galaxy SED model from radiative transfer
  templates (starburst, AGN torus, host galaxy, polar dust) for ultraviolet-to-millimetre
  fitting, e.g. with SMART or a reimplementation of its component-combination step.
when_to_use_negative:
- The analysis goal is energy-balance empirical SED fitting rather than fitting exclusively
  with radiative transfer models
- The data are outside the ultraviolet-to-millimetre SED domain or require components
  outside the four SMART library types (no radiative transfer template exists for
  them)
- Only a single-component fit is needed and the four-library composite model space
  is unnecessary overhead
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: SMART
  role: Provides the four pre-computed radiative transfer libraries (starburst, AGN
    torus, host galaxy, polar dust) whose templates are scaled and summed to assemble
    the model space for Bayesian MCMC SED fitting
  repo: https://github.com/ch-var/SMART.git
- name: Python 3.10
  role: Implementation environment (Anaconda package) for loading the library templates
    and implementing the component-combination function
- name: Spitzer
  role: Spectroscopy data source for the HERUS ULIRG SEDs fitted against the assembled
    model space
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/mnras/stae1141
    title: 'SMART: spectral energy distributions Markov chain analysis with radiative
      transfer models'
  source_content_hash: sha256:4977fc840a780059cf69ec08ab930eab30cb4fcca1917f956b72a7f3e98b9254
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ch-var/SMART.git
  tools:
  - Python 3.10
  - SMART
  - Spitzer
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# galaxy-sed-model-space-assembly

## Summary

Assembles the composite ultraviolet-to-millimetre galaxy SED model space used by SMART by scaling and summing templates drawn exclusively from four pre-computed radiative transfer libraries (starburst, AGN torus, host galaxy, polar dust) before Bayesian MCMC fitting.

## When to use

Use when building a multi-component galaxy SED model from radiative transfer templates (starburst, AGN torus, host galaxy, polar dust) for ultraviolet-to-millimetre fitting, e.g. with SMART or a reimplementation of its component-combination step.

## When NOT to use

- The analysis goal is energy-balance empirical SED fitting rather than fitting exclusively with radiative transfer models
- The data are outside the ultraviolet-to-millimetre SED domain or require components outside the four SMART library types (no radiative transfer template exists for them)
- Only a single-component fit is needed and the four-library composite model space is unnecessary overhead

## Inputs

- SMART v1.0 release archive (SMART_v1.0.7z) containing pre-computed radiative transfer model libraries
- User-specified library component selections (starburst, AGN torus, host galaxy, polar dust)
- SMART_User_Manual.pdf guidance on library formats
- Galaxy spectral energy distribution data spanning ultraviolet to millimetre wavelengths (e.g. Spitzer spectroscopy of HERUS ULIRGs)

## Outputs

- Composite multi-component ultraviolet-to-millimetre SED assembled from the four radiative transfer library types
- Scaled library template contributions per selected component

## How to apply

Obtain and extract the SMART v1.0 release archive (SMART_v1.0.7z, or SMART_v1.0_mac.7z on macOS), which contains the pre-computed radiative transfer model libraries, and load the four library types (starburst, AGN torus, host galaxy, polar dust) in Python 3.10 (Anaconda package), following SMART_User_Manual.pdf for library formats. Implement the component-combination function that scales each selected library template and sums the components to build a composite ultraviolet-to-millimetre SED, mirroring the model space SMART fits via Bayesian Markov chain Monte Carlo. Select which components to include per fit (e.g. omitting the AGN torus or polar dust for pure star-forming targets) and generate the composite multi-component SED from those user-specified selections. Validate by confirming the module runs end-to-end and that its output is a SED assembled exclusively from the four radiative transfer library types.

## Related tools

- **SMART** (Provides the four pre-computed radiative transfer libraries (starburst, AGN torus, host galaxy, polar dust) whose templates are scaled and summed to assemble the model space for Bayesian MCMC SED fitting) — https://github.com/ch-var/SMART.git
- **Python 3.10** (Implementation environment (Anaconda package) for loading the library templates and implementing the component-combination function)
- **Spitzer** (Spectroscopy data source for the HERUS ULIRG SEDs fitted against the assembled model space)

## Examples

```
python component_combination.py --components starburst,agn_torus,host_galaxy,polar_dust
```

## Evaluation signals

- The module runs end-to-end without error (script_runs check on component_combination.py)
- The generated composite SED file exists on disk (file_exists check)
- The composite output is a SED assembled exclusively from the four radiative transfer library types (starburst, AGN torus, host galaxy, polar dust) — no other component contributes flux
- The composite spans the ultraviolet-to-millimetre wavelength range expected of SMART model space

## Limitations

- The model space is restricted to the four currently available library types; components beyond starburst, AGN torus, host galaxy, and polar dust cannot be represented
- Windows/Linux users need SMART_v1.0.7z while macOS users need SMART_v1.0_mac.7z — the archives differ by platform
- Library formats must be loaded per SMART_User_Manual.pdf; no changelog is available for the repository

## Evidence

- [abstract] Model space assembled exclusively from four pre-computed radiative transfer libraries: "SMART fits the ultraviolet to millimetre SEDs of galaxies exclusively with radiative transfer models that currently constitute four types of pre-computed libraries"
- [abstract] Four library component types enumerated: "which describe the starburst, active galactic nucleus torus, host galaxy and polar dust components"
- [intro] Bayesian MCMC fitting method: [evidence span withheld — anchor status: not_found]
- [intro] SMART developed in Python 3.10: [evidence span withheld — anchor status: not_found]
- [readme] User manual is the documentation entry point for running SMART: "To learn how to run <b>SMART</b> and for a helpful user guide, please read the <a href="https://github.com/ch-var/SMART/blob/main/assets/SMART_User_Manual.pdf">manual</a>."
- [readme] Platform-specific release archives to download and extract: "please select your operating system below and then download (<img src="assets/download.png" width="18" height="18" />) the folder and extract its contents"
