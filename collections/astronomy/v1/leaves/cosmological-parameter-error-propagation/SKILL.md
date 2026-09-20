---
name: cosmological-parameter-error-propagation
description: Use when a Fisher-forecast-style error budget is needed for CMB or lensing
  power-spectrum data (e.g. SPT-3G TT/TE/EE, ACT phi-phi) at a fixed fiducial cosmology,
  and exact derivatives are preferred over finite differences.
when_to_use_negative:
- When full posterior constraints or non-Gaussian posteriors are needed — use gradient-based
  MCMC (e.g. BlackJAX) instead of a Fisher forecast at a fiducial point
- When the likelihood or theory pipeline is not differentiable/JAX-compatible (e.g.
  non-candl data sets requiring finite-difference derivatives)
- When priors must be excluded from the forecast and the setup relies on candl internal
  priors without adjusting clear_internal_priors after the default-behaviour change
tools:
- name: candl
  role: Differentiable likelihood providing candl.tools.get_fisher_matrix for JAX-based
    Fisher computation
  repo: github.com/Lbalkenhol/candl
- name: JAX
  role: Automatic differentiation backend computing exact gradients/Hessians of the
    likelihood, replacing finite differences
- name: CAMB
  role: Theory code via candl.interface.get_CAMB_pars_to_theory_specs_func mapping
    parameter dictionaries to CMB/lensing spectra
  repo: https://github.com/cmbant/CAMB
- name: CosmoPower-JAX
  role: Alternative differentiable emulator interface for the parameters-to-theory-specs
    function
  repo: https://github.com/dpiras/cosmopower-jax
- name: candl_data
  role: Supplies the data sets (SPT-3G 2018 TT/TE/EE, ACT DR6 Lens) initialised through
    candl.Like
  repo: https://github.com/Lbalkenhol/candl_data
provenance:
  source_task_ids:
  - task_008
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202449432
    title: 'candl: cosmic microwave background analysis with a differentiable likelihood'
  source_content_hash: sha256:70e3a1487f612b06d613e444f4926f3ec4e2bf5f84057c3d96b5f501ebe87723
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAMB
  - CosmoPower-JAX
  - JAX
  - Python
  - candl
  - candl_data
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cosmological-parameter-error-propagation

## Summary

Propagate experimental uncertainties to cosmological parameter errors by computing an exact Fisher matrix with JAX automatic differentiation of the candl CMB/lensing likelihood, inverting it to obtain 1-sigma parameter covariance without finite-difference derivatives.

## When to use

Use when a Fisher-forecast-style error budget is needed for CMB or lensing power-spectrum data (e.g. SPT-3G TT/TE/EE, ACT phi-phi) at a fixed fiducial cosmology, and exact derivatives are preferred over finite differences.

## When NOT to use

- When full posterior constraints or non-Gaussian posteriors are needed — use gradient-based MCMC (e.g. BlackJAX) instead of a Fisher forecast at a fiducial point
- When the likelihood or theory pipeline is not differentiable/JAX-compatible (e.g. non-candl data sets requiring finite-difference derivatives)
- When priors must be excluded from the forecast and the setup relies on candl internal priors without adjusting clear_internal_priors after the default-behaviour change

## Inputs

- candl likelihood (.yaml file or index-file entry, e.g. candl_data.ACT_DR6_Lens lens_only or candl_data.SPT3G_2018_TTTEEE)
- Theory-code interface mapping a parameter dictionary to CMB spectra (CAMB via candl.interface.get_CAMB_pars_to_theory_specs_func or CosmoPower-JAX)
- Fiducial parameter dictionary (e.g. H0=67.37, ombh2=0.02233, omch2=0.1198, logA=3.043, ns=0.9652, tau=0.054, yp=1.0)
- Optional parameter ordering (par_order) for the forecast

## Outputs

- Fisher matrix at the fiducial point (symmetric, from exact JAX derivatives)
- Parameter covariance matrix (Fisher inverse)
- 1-sigma parameter errors as sqrt(diag(cov)) with par_order labelling (e.g. written to act_dr6_pp_fisher.csv or bin_width_scan_results.csv)

## How to apply

Initialise the candl likelihood (candl.Like) from the relevant .yaml or index file (e.g. candl_data.ACT_DR6_Lens with lensing=True and variant='lens_only', or candl_data.SPT3G_2018_TTTEEE), and build a parameters-to-theory-specs function via candl.interface.get_CAMB_pars_to_theory_specs_func or the CosmoPower-JAX interface. Compose it with candl.tools.get_params_to_logl_func, keeping candl's internal priors so they contribute to the forecast (note: since the default-behaviour change, clear_internal_priors defaults to ignoring them — check the docs to keep them). Fix a Planck-18-style fiducial point (H0=67.37, ombh2=0.02233, omch2=0.1198, logA=3.043, ns=0.9652, tau=0.054, yp=1.0) and call candl.tools.get_fisher_matrix(pars_to_theory_specs, candl_like, fid_pars, par_order=None, return_par_order=True), which uses JAX-computed Hessians rather than finite differences. Invert the Fisher matrix to get the parameter covariance, take sqrt(diag) as 1-sigma errors, and label rows/columns with the returned par_order.

## Related tools

- **candl** (Differentiable likelihood providing candl.tools.get_fisher_matrix for JAX-based Fisher computation) — github.com/Lbalkenhol/candl
- **JAX** (Automatic differentiation backend computing exact gradients/Hessians of the likelihood, replacing finite differences)
- **CAMB** (Theory code via candl.interface.get_CAMB_pars_to_theory_specs_func mapping parameter dictionaries to CMB/lensing spectra) — https://github.com/cmbant/CAMB
- **CosmoPower-JAX** (Alternative differentiable emulator interface for the parameters-to-theory-specs function) — https://github.com/dpiras/cosmopower-jax
- **candl_data** (Supplies the data sets (SPT-3G 2018 TT/TE/EE, ACT DR6 Lens) initialised through candl.Like) — https://github.com/Lbalkenhol/candl_data

## Examples

```
par_cov, par_order = candl.tools.get_fisher_matrix(pars_to_theory_specs, candl_like, fid_pars, par_order=None, return_par_order=True)
```

## Evaluation signals

- Fisher matrix is symmetric and finite
- Output table lists one row per parameter in the returned par_order with finite, positive 1-sigma errors
- Derivatives come from exact JAX Hessians, not finite-difference approximations
- Errors are reported in the par_order returned by get_fisher_matrix (labels match matrix ordering)
- Internal priors (if intended to enter the forecast) are actually active after the clear_internal_priors default change

## Limitations

- Since v1.5-era default-behaviour change, candl internal priors are ignored by default; migration guidance lives in external docs, so priors may silently drop out of the Fisher forecast
- JAX on some architectures lacks 64-bit support, causing up to 1e-4 differences in log-likelihood values (noted for clipy; relevant to JAX-based derivative pipelines)
- Fisher forecasts are only valid near the fiducial point and assume Gaussian parameter posteriors; no archival (e.g. Zenodo) identifier for specific candl release versions is provided

## Evidence

- [abstract] robust Fisher matrices without finite-difference methods: "This facilitates, for example, robust Fisher matrices without finite-difference methods."
- [intro] JAX compatibility for gradients and Hessians: "JAX-compatibility, allowing for fast and easy computation of gradients and Hessians of the likelihoods."
- [intro] lensing power spectrum support (phi-phi, kappa-kappa): "candl supports the analysis of primary CMB and lensing power spectrum data"
- [methods] get_fisher_matrix usage pattern: "par_cov, par_order = candl.tools.get_fisher_matrix(pars_to_theory_specs, candl_like, fid_pars"
- [methods] parameters-to-log-likelihood composition: "like = candl.tools.get_params_to_logl_func(candl_like, pars_to_theory_specs)"
