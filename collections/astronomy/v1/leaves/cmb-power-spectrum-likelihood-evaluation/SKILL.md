---
name: cmb-power-spectrum-likelihood-evaluation
description: Use when a CMB TT/TE/EE, BB, lensing (phiphi, kappakappa) likelihood
  must be evaluated (or differentiated) at a set of cosmological and nuisance parameters,
  e.g. Planck/SPT/ACT band-power data with .yaml likelihood files.
when_to_use_negative:
- The task requires non-CMB likelihoods or data types not supported by candl (primary
  CMB and lensing power spectra TT/TE/EE/BB/phiphi/kappakappa only)
- Planck plik-style likelihood files must be used directly without conversion — use
  clipy (pure-Python JAX-compatible clik replacement) instead
- Only a coarse goodness-of-fit scan is needed where differentiability and JAX overheads
  add no value and a 32-bit JAX architecture could introduce ~1e-4 log-likelihood
  differences
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: candl
  role: 'Core differentiable likelihood code: initialises the likelihood from a .yaml
    index file, bins model spectra with window functions, applies data-model transformations,
    and computes the log-likelihood'
  repo: https://github.com/Lbalkenhol/candl
- name: JAX
  role: Powers candl's automatic differentiation, enabling fast computation of gradients
    and Hessians of the likelihood
  repo: https://github.com/google/jax
- name: CosmoPower-JAX
  role: Provides differentiable neural-network emulators (via candl.interface.get_CosmoPowerJAX_pars_to_theory_specs_func)
    mapping cosmological parameters to TT/TE/EE theory spectra
  repo: https://github.com/dpiras/cosmopower-jax
- name: clipy
  role: Pure-Python JAX-compatible implementation of Planck clik likelihoods, natively
    compatible with candl, for evaluating Planck likelihood files
  repo: https://github.com/benabed/clipy
- name: Cobaya
  role: Sampler integration via candl.interface.get_cobaya_info_dict_for_like to drive
    likelihood evaluations in MCMC runs
  repo: https://github.com/CobayaSampler/cobaya
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_002
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202449432
    title: 'candl: cosmic microwave background analysis with a differentiable likelihood'
  source_content_hash: sha256:6eaf2f192c7a986d41e97de579078e5a176f1540da661ff1090260e7b8d59e9b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lbalkenhol/candl
  tools:
  - BlackJAX
  - Cobaya
  - CosmoPower-JAX
  - JAX
  - Python
  - candl
  - clipy
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# cmb-power-spectrum-likelihood-evaluation

## Summary

Evaluate a CMB power spectrum likelihood with candl, a JAX-powered automatically differentiable Python likelihood, by initialising it from an index file (e.g. candl_data.SPT3G_2018_TTTEEE), passing model spectra through window-function binning and data-model transformations, and obtaining a log-likelihood value at a given parameter point.

## When to use

Use when a CMB TT/TE/EE, BB, lensing (phiphi, kappakappa) likelihood must be evaluated (or differentiated) at a set of cosmological and nuisance parameters, e.g. Planck/SPT/ACT band-power data with .yaml likelihood files.

## When NOT to use

- The task requires non-CMB likelihoods or data types not supported by candl (primary CMB and lensing power spectra TT/TE/EE/BB/phiphi/kappakappa only)
- Planck plik-style likelihood files must be used directly without conversion — use clipy (pure-Python JAX-compatible clik replacement) instead
- Only a coarse goodness-of-fit scan is needed where differentiability and JAX overheads add no value and a 32-bit JAX architecture could introduce ~1e-4 log-likelihood differences

## Inputs

- candl likelihood index file (.yaml), e.g. candl_data.SPT3G_2018_TTTEEE or candl_data.SPT3G_2018_Lens
- Dictionary of cosmological and nuisance parameters (e.g. H0, ombh2, omch2, logA, ns, tau, yp)
- Parameters-to-theory-spectra function from CosmoPower-JAX emulator files (cmb_spt_TT_NN, cmb_spt_TE_PCAplusNN, cmb_spt_EE_NN) or CAMB
- Optional data_selection dictionary specifying multipole cuts or band-power sub-selections

## Outputs

- Log-likelihood value at the given parameter point
- Differentiable parameters-to-log-likelihood function usable for gradients, Hessians, Fisher matrices, or NUTS sampling

## How to apply

Install candl (pip install candl-like) and the data repository (clone and pip install .), then initialise the likelihood by pointing to its .yaml index file, e.g. candl.Like(candl_data.SPT3G_2018_TTTEEE) for the SPT-3G 2018 TT/TE/EE release. Build a parameters-to-theory-spectra mapping with candl.interface (e.g. get_CosmoPowerJAX_pars_to_theory_specs_func using CosmoPower-JAX emulators such as cmb_spt_TT_NN, cmb_spt_TE_PCAplusNN, cmb_spt_EE_NN placed in cosmopower_jax/trained_models, or get_CAMB_pars_to_theory_specs_func for CAMB), and wrap everything with candl.tools.get_params_to_logl_func to obtain a single differentiable parameter-dictionary-to-log-likelihood function. Internally, candl bins the model spectra with the window functions, applies the data-model transformations (e.g. foreground, calibration and covariance transformations) one by one, and computes chi-squared from the data-model band-power difference. Evaluate at the fiducial point (Planck 18 best fit: H0=67.37, ombh2=0.02233, omch2=0.1198, logA=3.043, ns=0.9652, tau=0.054, yp=1.0) or any proposal point; because the function is JAX-differentiable, gradients and Hessians for Fisher matrices or gradient-based MCMC come for free. Filter the data content with the data_selection keyword (multipole cuts / band-power sub-selections) and toggle priors via clear_internal_priors or clear_specific_priors.

## Related tools

- **candl** (Core differentiable likelihood code: initialises the likelihood from a .yaml index file, bins model spectra with window functions, applies data-model transformations, and computes the log-likelihood) — https://github.com/Lbalkenhol/candl
- **JAX** (Powers candl's automatic differentiation, enabling fast computation of gradients and Hessians of the likelihood) — https://github.com/google/jax
- **CosmoPower-JAX** (Provides differentiable neural-network emulators (via candl.interface.get_CosmoPowerJAX_pars_to_theory_specs_func) mapping cosmological parameters to TT/TE/EE theory spectra) — https://github.com/dpiras/cosmopower-jax
- **clipy** (Pure-Python JAX-compatible implementation of Planck clik likelihoods, natively compatible with candl, for evaluating Planck likelihood files) — https://github.com/benabed/clipy
- **Cobaya** (Sampler integration via candl.interface.get_cobaya_info_dict_for_like to drive likelihood evaluations in MCMC runs) — https://github.com/CobayaSampler/cobaya

## Examples

```
import candl, candl_data; like = candl.Like(candl_data.SPT3G_2018_TTTEEE); logl = like.loglike(like.par_dict_to_fixed_CosmoPowerJAX_and_fid_pars(fid_pars))
```

## Evaluation signals

- Likelihood initialises from the .yaml index file without error and the internal pipeline runs: window-function binning, successive data-model transformations, chi-squared from the data-model band-power difference
- The log-likelihood evaluated at the Planck 18 best-fit point (H0=67.37, ombh2=0.02233, omch2=0.1198, logA=3.043, ns=0.9652, tau=0.054, yp=1.0) is finite and consistent with published values for SPT-3G 2018 TT/TE/EE
- Automatic-differentiation gradients and Hessians agree with central finite-difference estimates at the sub-percent level, consistent with the 0.1% tolerance candl uses for its numerical checks
- candl.tools.get_fisher_matrix returns a par_cov that is symmetric positive semi-definite with uncertainties consistent with the SPT-3G 2018 TT/TE/EE analysis
- data_selection sub-selections reproduce expected subset behaviour (e.g. a subset correlated with itself yields correlation 1)

## Limitations

- On some architectures JAX 32-bit computation truncates results, causing up to 1e-4 differences in log-likelihood values (noted for clipy plik files due to truncation of the Planck overall calibration)
- No archival (e.g. Zenodo) identifier is provided for specific candl release versions; compatibility is only stated for specific versions (e.g. JAX 0.4.31)
- A breaking change in default behaviour (candl internal priors now ignored, clear_internal_priors) requires checking external docs for migration guidance

## Evidence

- [abstract] We present candl, an automatically differentiable python likelihood for analysing cosmic microwave background power spectrum measurements.: "We present candl, an automatically differentiable python likelihood for analysing cosmic microwave background power spectrum measurements."
- [abstract] candl is powered by JAX: "candl is powered by JAX, which makes it fast and easy to calculate derivatives of the likelihood."
- [methods] It simply suffices to point to the .yaml file: "It simply suffices to point to the ``.yaml`` file of a likelihood to initialise it."
- [methods] Bin the model spectra using the window functions: "#. Bin the model spectra using the window functions."
- [methods] Take the difference between data and model band powers: "#. Take the difference between data and model band powers and calculate the log likelihood value."
- [methods] clipy, a pure Python implementation: "candl comes with ``clipy``, a pure Python implementation of the 2018 Planck likelihoods."
- [intro] pip install candl-like: "candl can be installed with pip::

    pip install candl-like"
