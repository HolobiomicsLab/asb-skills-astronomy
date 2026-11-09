---
name: jax-automatic-differentiation-derivatives
description: Use when you need parameter-derivative quantities from a CMB power-spectrum
  likelihood — Fisher matrices, parameter covariance, or gradient-based optimisation/sampling
  — and want exact derivatives rather than finite-difference estimates.
when_to_use_negative:
- 'The likelihood is not JAX-differentiable (e.g. clipy/candl running without JAX,
  or an external non-differentiable likelihood): autodiff is unavailable and finite
  differences would be needed'
- The analysis only needs a scalar log-likelihood evaluation (e.g. point-estimate
  chi-squared), for which the differentiation machinery is unnecessary overhead
- '32-bit-only JAX architectures where precision loss is unacceptable: clipy''s README
  notes up to 1e-4 differences in log-likelihood values on such platforms'
tools:
- name: candl
  role: Provides the automatically differentiable likelihood whose derivatives are
    taken; exposes candl.tools.get_fisher_matrix and candl.tools.get_params_to_logl_func
  repo: https://github.com/Lbalkenhol/candl
- name: JAX
  role: Autodiff engine that computes exact gradients and Hessians of the likelihood,
    enabling finite-difference-free Fisher matrices
  repo: https://github.com/google/jax
- name: CAMB
  role: Theory code wrapped via candl.interface.get_CAMB_pars_to_theory_specs_func
    to supply the differentiable theory-spectra mapping
  repo: https://github.com/cmbant/CAMB
- name: BlackJAX
  role: Gradient-based samplers that consume the same JAX log-likelihood derivatives
    for MCMC
  repo: https://github.com/blackjax-devs/blackjax
- name: Optax
  role: Gradient-based optimisation library used on the differentiable likelihood
  repo: https://github.com/google-deepmind/optax
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202449432
    title: 'candl: cosmic microwave background analysis with a differentiable likelihood'
  source_content_hash: sha256:285b3119e36f22023fb52edd6849065a04ced2b702d7a575486c5bd0b1817443
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lbalkenhol/candl
  tools:
  - BlackJAX
  - CAMB
  - JAX
  - Optax
  - Python
  - candl
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# jax-automatic-differentiation-derivatives

## Summary

Exploit JAX automatic differentiation through the candl differentiable CMB likelihood to obtain exact gradients and Hessians (e.g. Fisher matrices via candl.tools.get_fisher_matrix) without finite-difference approximations.

## When to use

Use when you need parameter-derivative quantities from a CMB power-spectrum likelihood — Fisher matrices, parameter covariance, or gradient-based optimisation/sampling — and want exact derivatives rather than finite-difference estimates.

## When NOT to use

- The likelihood is not JAX-differentiable (e.g. clipy/candl running without JAX, or an external non-differentiable likelihood): autodiff is unavailable and finite differences would be needed
- The analysis only needs a scalar log-likelihood evaluation (e.g. point-estimate chi-squared), for which the differentiation machinery is unnecessary overhead
- 32-bit-only JAX architectures where precision loss is unacceptable: clipy's README notes up to 1e-4 differences in log-likelihood values on such platforms

## Inputs

- candl.Like object initialised from a data-set .yaml file (e.g. candl_data.SPT3G_2018_TTTEEE or a mock data set)
- pars_to_theory_specs function mapping cosmological parameters to theory spectra (e.g. built with candl.interface.get_CAMB_pars_to_theory_specs_func)
- Fiducial parameter dictionary (H0, ombh2, omch2, logA, ns, tau, yp)
- Optional par_order list controlling the parameter ordering of derivatives

## Outputs

- Fisher matrix at the fiducial point (exact Hessian-based derivatives)
- Parameter covariance matrix (par_cov) from inverting the Fisher matrix
- Parameter ordering (par_order) returned alongside for consistent labelling
- 1-sigma parameter errors as square roots of the covariance diagonal

## How to apply

Initialise the likelihood with candl.Like from a data-set .yaml file (e.g. candl_data.SPT3G_2018_TTTEEE), then build a parameters-to-theory function such as candl.interface.get_CAMB_pars_to_theory_specs_func so that cosmological parameters flow through the theory pipeline into the binned band powers. Fix a fiducial point (in the paper's examples: H0=67.37, ombh2=0.02233, omch2=0.1198, logA=3.043, ns=0.9652, tau=0.054, yp=1.0) and call candl.tools.get_fisher_matrix(pars_to_theory_specs, candl_like, fid_pars, par_order=None, return_par_order=True); JAX differentiates the log-likelihood exactly, yielding the Hessian-based Fisher matrix and the returned par_order for consistent parameter labelling. Invert the Fisher matrix to get the parameter covariance (par_cov) and take 1-sigma errors as the square roots of its diagonal. The rationale is that JAX-compatibility 'allows for fast and easy computation of gradients and Hessians of the likelihoods', giving 'robust Fisher matrices without finite-difference methods', so results are exact and step-size-free. Reuse the same differentiable log-likelihood (e.g. via candl.tools.get_params_to_logl_func) for gradient-based MCMC (BlackJAX) or gradient-based optimisation (Optax).

## Related tools

- **candl** (Provides the automatically differentiable likelihood whose derivatives are taken; exposes candl.tools.get_fisher_matrix and candl.tools.get_params_to_logl_func) — https://github.com/Lbalkenhol/candl
- **JAX** (Autodiff engine that computes exact gradients and Hessians of the likelihood, enabling finite-difference-free Fisher matrices) — https://github.com/google/jax
- **CAMB** (Theory code wrapped via candl.interface.get_CAMB_pars_to_theory_specs_func to supply the differentiable theory-spectra mapping) — https://github.com/cmbant/CAMB
- **BlackJAX** (Gradient-based samplers that consume the same JAX log-likelihood derivatives for MCMC) — https://github.com/blackjax-devs/blackjax
- **Optax** (Gradient-based optimisation library used on the differentiable likelihood) — https://github.com/google-deepmind/optax

## Examples

```
par_cov, par_order = candl.tools.get_fisher_matrix(pars_to_theory_specs, candl_like, fid_pars, par_order=None, return_par_order=True)
```

## Evaluation signals

- candl.tools.get_fisher_matrix returns return_par_order=True output: check the covariance dimension matches len(par_order)
- Verify no finite differences are involved: derivative values must be reproducible exactly (no step-size dependence), consistent with 'robust Fisher matrices without finite-difference methods'
- Invertibility check: the Fisher matrix should be positive-definite so the parameter covariance exists; 1-sigma errors are the square roots of the diagonal
- Cross-check a derivative against a finite-difference or numerical-Hessian estimate as a sanity test (small discrepancy expected, since JAX is exact)
- On JAX architectures without 64-bit support, expect log-likelihood differences up to ~1e-4 (per the clipy README) — flag results computed in 32-bit mode

## Limitations

- JAX on some architectures does not support 64-bit computation, causing up to 1e-4 differences in log-likelihood values (e.g. from truncation of the Planck overall calibration in plik files)
- The whole chain from parameters to theory spectra must be JAX-differentiable (e.g. via CAMB wrappers); non-differentiable links break the exact-Hessian guarantee
- Version compatibility matters: the changelog states compatibility with JAX version 0.4.31 (tested with TF 2.17.0), so derivative behaviour should be revalidated on other versions
- Since v1.0.1+, default behaviour changed so candl internal priors are ignored — derivative-based forecasts must pass priors explicitly or results will differ

## Evidence

- [abstract] JAX powers candl autodiff: "candl is powered by JAX, which makes it fast and easy to calculate derivatives of the likelihood."
- [abstract] Robust Fisher matrices without finite differences: "This facilitates, for example, robust Fisher matrices without finite-difference methods."
- [methods] get_fisher_matrix API: "par_cov, par_order = candl.tools.get_fisher_matrix(pars_to_theory_specs, candl_like, fid_pars"
- [intro] Gradients and Hessians via JAX compatibility: "JAX-compatibility, allowing for fast and easy computation of gradients and Hessians of the likelihoods."
- [readme] clipy JAX support and 32-bit caveat: [evidence span withheld — anchor status: not_found]
