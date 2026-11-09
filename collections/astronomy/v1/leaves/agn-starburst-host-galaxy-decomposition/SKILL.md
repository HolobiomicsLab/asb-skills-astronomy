---
name: agn-starburst-host-galaxy-decomposition
description: Use when a galaxy ultraviolet-to-millimetre SED must be decomposed into
  or reconstructed from SMART's starburst, AGN torus, host galaxy, and polar dust
  radiative transfer library components.
when_to_use_negative:
- The analysis requires non-radiative-transfer SED models — SMART fits exclusively
  with its four pre-computed radiative transfer libraries and cannot incorporate other
  model families.
- The input is not a galaxy SED (UV-to-mm fluxes/spectra) but, e.g., a stellar-population-only
  or already-decomposed component table.
- The environment cannot run Python 3.10 (Anaconda package), the only version in which
  the code is developed.
tools:
- name: SMART
  role: Open-source SED fitting code supplying the four pre-computed radiative transfer
    model libraries and the Bayesian MCMC fitting that consumes the composite model
    space
  repo: https://github.com/ch-var/SMART.git
- name: Python 3.10
  role: Runtime environment (Anaconda package) in which the libraries are loaded and
    component_combination.py is implemented and executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/mnras/stae1141
    title: 'SMART: spectral energy distributions Markov chain analysis with radiative
      transfer models'
  source_content_hash: sha256:c30bc15e5d511189deadf8030ec660872b8b1ff51c1a3185b41dc8b22f8e0ab8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ch-var/SMART.git
  tools:
  - Python 3.10
  - SMART
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# agn-starburst-host-galaxy-decomposition

## Summary

Reconstructs SMART's model space by combining its four types of pre-computed radiative transfer model libraries — starburst, AGN torus, host galaxy, and polar dust — into composite ultraviolet-to-millimetre galaxy SEDs, as fitted via Bayesian MCMC.

## When to use

Use when a galaxy ultraviolet-to-millimetre SED must be decomposed into or reconstructed from SMART's starburst, AGN torus, host galaxy, and polar dust radiative transfer library components.

## When NOT to use

- The analysis requires non-radiative-transfer SED models — SMART fits exclusively with its four pre-computed radiative transfer libraries and cannot incorporate other model families.
- The input is not a galaxy SED (UV-to-mm fluxes/spectra) but, e.g., a stellar-population-only or already-decomposed component table.
- The environment cannot run Python 3.10 (Anaconda package), the only version in which the code is developed.

## Inputs

- SMART v1.0 release archive (SMART_v1.0.7z or SMART_v1.0_mac.7z) containing pre-computed radiative transfer model libraries
- User-specified library component selections (starburst, AGN torus, host galaxy, polar dust)
- Multiwavelength ultraviolet-to-millimetre photometry and/or spectroscopy (e.g. Spitzer spectroscopy of HERUS ULIRGs)
- SMART_User_Manual.pdf for library format and run guidance

## Outputs

- Composite multi-component ultraviolet-to-millimetre SED assembled from the four radiative transfer library types
- component_combination.py module performing template scaling and summation
- Fitted SED parameters from SMART's Bayesian MCMC when applied to observed data

## How to apply

Obtain and extract the SMART v1.0 release archive (SMART_v1.0.7z for Windows/Linux, SMART_v1.0_mac.7z for macOS), which contains the pre-computed radiative transfer model libraries. In Python 3.10 (Anaconda package), load the four library types — starburst, AGN torus, host galaxy, and polar dust — following the library-format guidance in SMART_User_Manual.pdf. Implement a component-combination function (component_combination.py) that scales each selected library template and sums the components to build a composite UV-to-mm SED, mirroring the model space SMART fits with its Bayesian Markov chain Monte Carlo method. Spitzer spectroscopy (e.g. of the 42 HERUS ULIRGs) can be included in the fitting at a resolution matched to the radiative transfer models.

## Related tools

- **SMART** (Open-source SED fitting code supplying the four pre-computed radiative transfer model libraries and the Bayesian MCMC fitting that consumes the composite model space) — https://github.com/ch-var/SMART.git
- **Python 3.10** (Runtime environment (Anaconda package) in which the libraries are loaded and component_combination.py is implemented and executed)

## Examples

```
python component_combination.py  # scale and sum selected starburst, AGN torus, host galaxy, polar dust templates into a composite UV-to-mm SED
```

## Evaluation signals

- component_combination.py runs end-to-end without error (script_runs check)
- The generated composite output exists on disk (file_exists check)
- The composite SED is assembled exclusively from the four radiative transfer library types (starburst, AGN torus, host galaxy, polar dust) with no other components
- Each selected library template is scaled before summation, and the output spans the ultraviolet-to-millimetre range
- When fitting observed data (e.g. HERUS ULIRGs with Spitzer spectroscopy), spectroscopy resolution is matched to the radiative transfer models

## Limitations

- SMART fits SEDs exclusively with radiative transfer models; energy-balance or empirical SED models are outside its model space.
- The code is developed in PYTHON 3.10 (Anaconda package); other Python versions are untested per the article/README.
- The README notes no changelog is available, so version-to-version library changes cannot be tracked from the release metadata.

## Evidence

- [readme] SMART fits UV-to-mm SEDs exclusively with RT libraries: "fits the ultraviolet to millimetre SEDs of galaxies exclusively with radiative transfer models that currently constitute four types of pre-computed libraries"
- [abstract] SMART is developed in Python and available on GitHub: "SMART is developed in Python and is available at https://github.com/ch-var/SMART.git"
- [intro] Python 3.10 Anaconda development environment: [evidence span withheld — anchor status: not_found]
- [readme] Bayesian MCMC fitting method: "Implementing a Bayesian Markov chain Monte Carlo method,"
- [readme] GPL-3.0 license terms: "SMART is a free tool made available under the terms of the"
