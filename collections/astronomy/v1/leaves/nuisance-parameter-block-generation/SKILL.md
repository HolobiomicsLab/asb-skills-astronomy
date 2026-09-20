---
name: nuisance-parameter-block-generation
description: Use when embedding a candl CMB power spectrum likelihood (SPT/ACT/Planck
  band-power data in .yaml info files) into a MontePython run, and you need the nuisance
  parameters to declare in the .param file.
when_to_use_negative:
- When sampling the likelihood with Cobaya instead of MontePython — candl provides
  a separate interface, candl.interface.get_cobaya_info_dict_for_like.
- When you want candl's internal priors to be applied by candl itself — clear_internal_priors
  must not be set True in candl_mp.data in that case.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: candl
  role: Source of the interface utility get_montepython_nuisance_param_block_for_like
    and of the Like object whose logl() provides the validation reference
  repo: https://github.com/Lbalkenhol/candl
- name: MontePython
  role: Sampling framework into which the candl_mp wrapper and generated nuisance
    parameter block are installed
- name: pip
  role: Installation channel for candl (pip install candl-like) and the candl data
    repositories (pip install . inside cloned repo)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1051/0004-6361/202449432
    title: 'candl: cosmic microwave background analysis with a differentiable likelihood'
  source_content_hash: sha256:a29d08d86cf5c03d8ab7a7b1a84ec9871efbeb70fd249183ab887c62dd286e47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lbalkenhol/candl
  tools:
  - MontePython
  - Python
  - candl
  - pip
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# nuisance-parameter-block-generation

## Summary

Generates the MontePython nuisance parameter block for a candl CMB likelihood using candl.interface.get_montepython_nuisance_param_block_for_like, so a candl likelihood (e.g. candl_data.SPT3G_2018_TTTEEE) can be sampled inside MontePython with priors owned by MontePython rather than candl.

## When to use

Use when embedding a candl CMB power spectrum likelihood (SPT/ACT/Planck band-power data in .yaml info files) into a MontePython run, and you need the nuisance parameters to declare in the .param file.

## When NOT to use

- When sampling the likelihood with Cobaya instead of MontePython — candl provides a separate interface, candl.interface.get_cobaya_info_dict_for_like.
- When you want candl's internal priors to be applied by candl itself — clear_internal_priors must not be set True in candl_mp.data in that case.

## Inputs

- Initialised candl.Like object (e.g. candl_data.SPT3G_2018_TTTEEE index file or a .yaml info file path)
- MontePython likelihood wrapper folder candl_mp (__init__.py plus candl_mp.data) placed in montepython/likelihoods
- candl_mp.data options file with data set, lensing, feedback, and clear_internal_priors settings
- MontePython .param file awaiting the nuisance parameter block

## Outputs

- Nuisance parameter block (from candl.interface.get_montepython_nuisance_param_block_for_like) pasted into the .param file
- Paired log-likelihood values (MontePython wrapper vs direct candl logl) with absolute and relative differences recorded in candl_mp_logl_check.txt

## How to apply

Initialise the target candl.Like object on its data set — e.g. candl_data.SPT3G_2018_TTTEEE or a path to its .yaml file — then call candl.interface.get_montepython_nuisance_param_block_for_like on it to obtain the ready-made nuisance parameter block, and paste that block into the MontePython .param file. Install the wrapper (candl_mp: __init__.py plus a candl_mp.data options file) into montepython/likelihoods, with candl_mp.data setting the fixed data set, lensing = False, feedback = True, and clear_internal_priors = True so MontePython owns the priors instead of candl (v1.x changed defaults to ignore candl internal priors, so this flag must be explicit). Validate by evaluating the log-likelihood through MontePython at a fixed parameter point and comparing it to a direct candl.Like(...).logl() call on the identical data set and parameter values.

## Related tools

- **candl** (Source of the interface utility get_montepython_nuisance_param_block_for_like and of the Like object whose logl() provides the validation reference) — https://github.com/Lbalkenhol/candl
- **MontePython** (Sampling framework into which the candl_mp wrapper and generated nuisance parameter block are installed)
- **pip** (Installation channel for candl (pip install candl-like) and the candl data repositories (pip install . inside cloned repo))

## Examples

```
import candl; like = candl.Like('candl_data.SPT3G_2018_TTTEEE'); block = candl.interface.get_montepython_nuisance_param_block_for_like(like)
```

## Evaluation signals

- MontePython-evaluated log-likelihood and direct candl.Like(...).logl() at the same fixed parameter point agree within 0.1% relative difference (candl's own likelihood-test pass criterion)
- candl_mp_logl_check.txt records both log-likelihood values, their absolute difference, and the relative difference
- candl_mp.data sets clear_internal_priors = True, confirming MontePython, not candl, owns the priors
- The generated nuisance parameter block runs without MontePython complaining about undeclared parameters for the chosen likelihood

## Limitations

- Since candl v1.x the default behaviour ignores candl internal priors, so prior ownership must be set explicitly (clear_internal_priors) when constructing candl_mp.data.
- The validation tolerates only 0.1% relative difference; JAX 32-bit truncation on some architectures can introduce up to 1e-4 differences in log-likelihood computations (noted for clipy, a related JAX-based likelihood).
- candl_mp.data expects a fixed data set name or .yaml path; lensing must be disabled (lensing = False) in this wrapper configuration.

## Evidence

- [other] candl's interface module provides a dedicated MontePython utility, candl.interface.get_montepython_nuisance_param_block_for_like, which supplies the nuisance parameter block needed to run a candl likelihood under MontePython.: [evidence span withheld — anchor status: not_found]
- [other] clear_internal_priors = True so MontePython, not candl, owns the priors.: [evidence span withheld — anchor status: not_found]
- [other] Validation: the wrapper and direct-candl log-likelihood values must agree within 0.1% relative difference.: [evidence span withheld — anchor status: not_found]
- [intro] Interface tools for work with other popular cosmology software packages (e.g. Cobaya and MontePython).: "Interface tools for work with other popular cosmology software packages (e.g. Cobaya and MontePython)."
- [methods] clear_internal_priors: True # Switch off to use candl internal priors: "clear_internal_priors: True # Switch off to use candl internal priors"
- [discussion] Change in default behaviour: candl internal priors now ignored. Check docs for how to keep them: "__NOTE:__ Change in default behaviour: candl internal priors now ignored. Check docs for how to keep them"
