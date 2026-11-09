---
name: mock-data-set-likelihood-initialisation
description: Use when a realistic mock CMB band power data set (TT/TE/EE/BB/phi-phi/kappa-kappa)
  must be turned into a working differentiable likelihood for Fisher forecasting,
  e.g. scanning band power bin widths to minimise parameter errors.
when_to_use_negative:
- The analysis target is not CMB power-spectrum data (TT, TE, EE, BB, phi-phi, kappa-kappa)
  — candl is scoped to CMB band power measurements
- You need MCMC posterior sampling rather than Fisher forecasts — use the gradient-based
  sampler interfaces (Cobaya/MontePython/BlackJAX) instead of direct Fisher construction
- The likelihood must reproduce Planck clik files with legacy semantics — use clipy
  (the pure-Python clik replacement) rather than reinitialising from a candl .yaml
tools:
- name: candl
  role: differentiable likelihood framework; candl.Like initialised from .yaml and
    candl.tools.get_fisher_matrix computes Fisher matrices with JAX derivatives
  repo: https://github.com/Lbalkenhol/candl
- name: JAX
  role: powers automatic differentiation of the likelihood for exact gradients/Hessians
    (robust Fisher matrices without finite differences)
  repo: https://github.com/google/jax
- name: CAMB
  role: theory backend wired to the likelihood via candl.interface.get_CAMB_pars_to_theory_specs_func
    to map cosmological parameters to model spectra
  repo: https://github.com/cmbant/CAMB
- name: Python
  role: implementation language for the likelihood initialisation and Fisher scanning
    scripts
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202449432
    title: 'candl: cosmic microwave background analysis with a differentiable likelihood'
  source_content_hash: sha256:d8cee01772db348f30d4fd6358a23aa588aa8e585500b7f5e0454862f7f74264
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lbalkenhol/candl
  tools:
  - CAMB
  - JAX
  - Python
  - candl
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# mock-data-set-likelihood-initialisation

## Summary

Initialise a candl likelihood (candl.Like) directly from a mock CMB power-spectrum data set's .yaml file, bin the model spectra with window functions, and use the JAX-differentiable likelihood as the basis for exact Fisher forecasts (e.g. band power bin width optimisation) on that mock data set.

## When to use

Use when a realistic mock CMB band power data set (TT/TE/EE/BB/phi-phi/kappa-kappa) must be turned into a working differentiable likelihood for Fisher forecasting, e.g. scanning band power bin widths to minimise parameter errors.

## When NOT to use

- The analysis target is not CMB power-spectrum data (TT, TE, EE, BB, phi-phi, kappa-kappa) — candl is scoped to CMB band power measurements
- You need MCMC posterior sampling rather than Fisher forecasts — use the gradient-based sampler interfaces (Cobaya/MontePython/BlackJAX) instead of direct Fisher construction
- The likelihood must reproduce Planck clik files with legacy semantics — use clipy (the pure-Python clik replacement) rather than reinitialising from a candl .yaml

## Inputs

- mock data set likelihood .yaml file (e.g. from SouthPoleTelescope/spt_candl_forecasts)
- window functions for binning model spectra
- fiducial cosmological parameter dictionary (e.g. H0=67.37, ombh2=0.02233, omch2=0.1198, logA=3.043, ns=0.9652, tau=0.054, yp=1.0)
- pars_to_theory_specs mapping (e.g. from candl.interface.get_CAMB_pars_to_theory_specs_func)

## Outputs

- initialised candl.Like likelihood object
- binned model band powers and log-likelihood values
- Fisher matrix and parameter covariance (par_cov) with par_order labels
- 1-sigma parameter errors (square roots of covariance diagonal)

## How to apply

Initialise the likelihood by pointing candl to the mock data set's .yaml file — no further setup is needed for candl to load band powers, covariance, and window functions. Bin the model spectra using the window functions and loop over data-model transformations successively before computing the log-likelihood. For forecasting, wire the likelihood to a theory code via candl.interface.get_CAMB_pars_to_theory_specs_func, fix a fiducial cosmological point (e.g. H0=67.37, ombh2=0.02233, omch2=0.1198, logA=3.043, ns=0.9652, tau=0.054, yp=1.0), and compute exact Fisher matrices with candl.tools.get_fisher_matrix(pars_to_theory_specs, candl_like, fid_pars, return_par_order=True), exploiting JAX automatic differentiation of gradients/Hessians rather than finite differences. When scanning bin widths, rebin the band powers via the window functions for each candidate width, invert each Fisher matrix, and take 1-sigma errors as the square roots of the covariance diagonal, using the returned par_order for consistent parameter labelling. This supports candl's own example of optimising the band power bin width to minimise parameter errors of a realistic mock data set.

## Related tools

- **candl** (differentiable likelihood framework; candl.Like initialised from .yaml and candl.tools.get_fisher_matrix computes Fisher matrices with JAX derivatives) — https://github.com/Lbalkenhol/candl
- **JAX** (powers automatic differentiation of the likelihood for exact gradients/Hessians (robust Fisher matrices without finite differences)) — https://github.com/google/jax
- **CAMB** (theory backend wired to the likelihood via candl.interface.get_CAMB_pars_to_theory_specs_func to map cosmological parameters to model spectra) — https://github.com/cmbant/CAMB
- **Python** (implementation language for the likelihood initialisation and Fisher scanning scripts)

## Examples

```
candl_like = candl.Like('mock_data_set.yaml'); par_cov, par_order = candl.tools.get_fisher_matrix(pars_to_theory_specs, candl_like, fid_pars, return_par_order=True)
```

## Evaluation signals

- Likelihood initialises with no code beyond pointing to the .yaml file of the data set
- Fisher matrices are obtained from exact JAX derivatives (Hessians), not finite-difference approximations
- Parameter errors match the square roots of the diagonal of the inverted Fisher matrix, labelled consistently by the returned par_order
- For a bin-width scan, the reported optimal bin width is the scanned grid point at which the minimum parameter error is attained
- Priors: verify whether candl internal priors were applied or cleared (clear_internal_priors), since default behaviour changed in recent versions

## Limitations

- Recent candl versions changed default behaviour so candl internal priors are now ignored by default; migration guidance is deferred to external docs
- On some architectures JAX only offers 32-bit computation, producing up to 10^-4 differences in log-likelihood values (noted in the clipy README)
- No DOI or archival (e.g. Zenodo) identifier is provided for specific software release versions, so pinning exact versions requires care

## Evidence

- [methods] It simply suffices to point to the ``.yaml`` file of a likelihood to initialise it.: "It simply suffices to point to the ``.yaml`` file of a likelihood to initialise it."
- [abstract] These also include optimising the band power bin width to minimise parameter errors of a realistic mock data set.: "These also include optimising the band power bin width to minimise parameter errors of a realistic mock data set."
- [abstract] This facilitates, for example, robust Fisher matrices without finite-difference methods.: "This facilitates, for example, robust Fisher matrices without finite-difference methods."
- [methods] Bin the model spectra using the window functions.: "#. Bin the model spectra using the window functions."
- [intro] JAX-compatibility, allowing for fast and easy computation of gradients and Hessians of the likelihoods.: "JAX-compatibility, allowing for fast and easy computation of gradients and Hessians of the likelihoods."
