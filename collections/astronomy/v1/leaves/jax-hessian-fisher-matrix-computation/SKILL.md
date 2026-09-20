---
name: jax-hessian-fisher-matrix-computation
description: Use when you need parameter forecasts or uncertainty estimates from a
  CMB power spectrum likelihood (e.g. SPT-3G 2018 TT/TE/EE) and want robust derivatives
  without finite-difference step-size tuning.
when_to_use_negative:
- When the theory model is not JAX-differentiable (e.g. non-emulator Boltzmann pipelines
  without a JAX interface) — derivatives would silently fail or fall back to non-differentiable
  code paths
- When you need full posterior inference rather than Gaussian Fisher forecasts — use
  gradient-based MCMC (e.g. BlackJAX) instead
- When the likelihood relies on external non-differentiable code (e.g. fortran clik);
  note clipy replaces clik with a pure-Python JAX-compatible implementation for Planck-like
  cases
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_3360
- http://edamontology.org/topic_3452
tools:
- name: candl
  role: Provides the differentiable CMB likelihood and candl.tools.get_fisher_matrix,
    which differentiates the likelihood via JAX Hessians
  repo: https://github.com/Lbalkenhol/candl
- name: JAX
  role: Backend that computes exact gradients and Hessians of the log-likelihood,
    replacing finite-difference derivatives
- name: CosmoPower-JAX
  role: JAX-based emulator supplying the parameters-to-theory-spectra mapping for
    TT, TE, EE spectra of SPT-3G 2018
  repo: https://github.com/dpiras/cosmopower-jax
- name: spt_candl_data
  role: Official SPT data repository providing the SPT-3G 2018 TT/TE/EE likelihood
    index file
  repo: https://github.com/SouthPoleTelescope/spt_candl_data
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1051/0004-6361/202449432
    title: 'candl: cosmic microwave background analysis with a differentiable likelihood'
  source_content_hash: sha256:9bdfd96166115e3b775bdb95eb2175666b801247327d75781dc80e659b7210cb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lbalkenhol/candl
  tools:
  - CosmoPower-JAX
  - JAX
  - Python
  - candl
  - spt_candl_data
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_3360
  - http://edamontology.org/topic_3452
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# jax-hessian-fisher-matrix-computation

## Summary

Compute Fisher parameter covariance matrices for CMB power spectrum analyses by differentiating the candl likelihood with JAX Hessians, eliminating finite-difference approximations entirely.

## When to use

Use when you need parameter forecasts or uncertainty estimates from a CMB power spectrum likelihood (e.g. SPT-3G 2018 TT/TE/EE) and want robust derivatives without finite-difference step-size tuning.

## When NOT to use

- When the theory model is not JAX-differentiable (e.g. non-emulator Boltzmann pipelines without a JAX interface) — derivatives would silently fail or fall back to non-differentiable code paths
- When you need full posterior inference rather than Gaussian Fisher forecasts — use gradient-based MCMC (e.g. BlackJAX) instead
- When the likelihood relies on external non-differentiable code (e.g. fortran clik); note clipy replaces clik with a pure-Python JAX-compatible implementation for Planck-like cases

## Inputs

- candl likelihood index file (e.g. candl_data.SPT3G_2018_TTTEEE)
- CosmoPower-JAX trained emulator files (cmb_spt_TT_NN, cmb_spt_TE_PCAplusNN, cmb_spt_EE_NN) in cosmopower_jax/trained_models
- Fiducial cosmological parameter dictionary (H0, ombh2, omch2, logA, ns, tau, yp)
- Parameters-to-log-likelihood function built with candl.tools.get_params_to_logl_func

## Outputs

- Parameter covariance matrix (par_cov) from the Fisher/Hessian calculation
- Parameter ordering list (par_order) matching rows/columns of the covariance matrix

## How to apply

Initialise the likelihood with candl.Like(candl_data.SPT3G_2018_TTTEEE) pointing at the SPT-3G 2018 TT/TE/EE index file. Build a parameters-to-theory-spectra mapping with candl.interface.get_CosmoPowerJAX_pars_to_theory_specs_func using the CosmoPower SPT emulators {TT: cmb_spt_TT_NN, TE: cmb_spt_TE_PCAplusNN, EE: cmb_spt_EE_NN} placed in cosmopower_jax/trained_models. Assemble the differentiable parameters-to-log-likelihood function via candl.tools.get_params_to_logl_func(candl_like, pars_to_theory_specs). Define the fiducial parameter dictionary (H0: 67.37, ombh2: 0.02233, omch2: 0.1198, logA: 3.043, ns: 0.9652, tau: 0.054, yp: 1.0) and call candl.tools.get_fisher_matrix(pars_to_theory_specs, candl_like, fid_pars, par_order=None, return_par_order=True); JAX differentiates the likelihood through exact Hessians instead of finite differences, avoiding step-size and truncation-error artifacts.

## Related tools

- **candl** (Provides the differentiable CMB likelihood and candl.tools.get_fisher_matrix, which differentiates the likelihood via JAX Hessians) — https://github.com/Lbalkenhol/candl
- **JAX** (Backend that computes exact gradients and Hessians of the log-likelihood, replacing finite-difference derivatives)
- **CosmoPower-JAX** (JAX-based emulator supplying the parameters-to-theory-spectra mapping for TT, TE, EE spectra of SPT-3G 2018) — https://github.com/dpiras/cosmopower-jax
- **spt_candl_data** (Official SPT data repository providing the SPT-3G 2018 TT/TE/EE likelihood index file) — https://github.com/SouthPoleTelescope/spt_candl_data

## Examples

```
import candl, candl_data; candl_like = candl.Like(candl_data.SPT3G_2018_TTTEEE); par_cov, par_order = candl.tools.get_fisher_matrix(pars_to_theory_specs, candl_like, fid_pars, par_order=None, return_par_order=True)
```

## Evaluation signals

- The returned parameter covariance matrix (par_cov) must be symmetric positive semi-definite
- Square-root diagonal entries of par_cov must give parameter uncertainties consistent with those quoted for the SPT-3G 2018 TT/TE/EE analysis
- par_order returned with return_par_order=True must match the row/column ordering of the covariance matrix
- No finite-difference step-size parameters should appear or be needed in the call — the Hessian is computed exactly by JAX autodiff

## Limitations

- On some architectures, 64-bit computation is unavailable in JAX, causing log-likelihood differences of up to 1e-4 (documented in the clipy README for JAX-compatible likelihoods)
- candl compatibility has only been stated as tested with a single JAX version (0.4.31 / TF 2.17.0), so version drift may break the differentiable pipeline
- The Fisher result is a Gaussian approximation; the article notes it facilitates forecasts and robustness tests, not full non-Gaussian posteriors

## Evidence

- [abstract] Fisher via JAX Hessians: "This facilitates, for example, robust Fisher matrices without finite-difference methods."
- [methods] get_fisher_matrix call: "par_cov, par_order = candl.tools.get_fisher_matrix(pars_to_theory_specs, candl_like, fid_pars"
- [intro] Hessian computation via JAX: "JAX-compatibility, allowing for fast and easy computation of gradients and Hessians of the likelihoods."
- [readme] JAX Fisher use in clipy README: [evidence span withheld — anchor status: external_doc]
