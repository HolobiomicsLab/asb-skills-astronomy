---
name: fisher-matrix-forecasting
description: Use when you need 1-sigma parameter error forecasts (e.g. sigma(H0),
  sigma(ombh2)) from a CMB power-spectrum likelihood without running MCMC, or when
  scanning an experimental design choice (e.g. bin width) against forecast precision.
when_to_use_negative:
- You need full posterior constraints or non-Gaussian degeneracies — use gradient-based
  MCMC sampling (e.g. BlackJAX) instead of the Gaussian Fisher approximation
- The likelihood is not JAX-differentiable or derivatives must come from finite differences
  — candl's Fisher route assumes exact automatic differentiation of the likelihood
- The data set cannot be rebinned via its window functions (e.g. unbinned Planck plik
  files handled through clipy/clik) — bin-width scanning presumes window-function
  rebinning support
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3399
- http://edamontology.org/topic_0102
tools:
- name: candl
  role: Differentiable CMB likelihood providing candl.Like initialisation from .yaml
    files and candl.tools.get_fisher_matrix for Fisher forecasting
  repo: https://github.com/Lbalkenhol/candl
- name: JAX
  role: Automatic differentiation backend supplying exact gradients and Hessians of
    the likelihood, avoiding finite differences
  repo: https://github.com/google/jax
- name: Python
  role: Implementation language for the bin-width scan loop and error tabulation
- name: CAMB
  role: Theory provider via candl.interface.get_CAMB_pars_to_theory_specs_func to
    map parameters to CMB spectra
  repo: https://github.com/cmbant/CAMB
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202449432
    title: 'candl: cosmic microwave background analysis with a differentiable likelihood'
  source_content_hash: sha256:c75e7fd8d8eb24c786ce222d6b72cdc2e02e749a926310d5e403d5ac673b2af5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lbalkenhol/candl
  tools:
  - CAMB
  - JAX
  - Python
  - candl
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3399
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# fisher-matrix-forecasting

## Summary

Compute parameter error forecasts via Fisher matrices using candl's differentiable likelihood: exact Hessians from JAX automatic differentiation replace finite-difference methods, enabling scans such as optimising the band power bin width to minimise parameter errors on a realistic mock data set.

## When to use

Use when you need 1-sigma parameter error forecasts (e.g. sigma(H0), sigma(ombh2)) from a CMB power-spectrum likelihood without running MCMC, or when scanning an experimental design choice (e.g. bin width) against forecast precision.

## When NOT to use

- You need full posterior constraints or non-Gaussian degeneracies — use gradient-based MCMC sampling (e.g. BlackJAX) instead of the Gaussian Fisher approximation
- The likelihood is not JAX-differentiable or derivatives must come from finite differences — candl's Fisher route assumes exact automatic differentiation of the likelihood
- The data set cannot be rebinned via its window functions (e.g. unbinned Planck plik files handled through clipy/clik) — bin-width scanning presumes window-function rebinning support

## Inputs

- candl likelihood .yaml file for a mock or real CMB power-spectrum data set (e.g. from spt_candl_data or candl_data)
- Fiducial cosmological parameter dictionary (e.g. H0, ombh2, omch2, logA, ns, tau, yp)
- pars_to_theory_specs function mapping cosmological parameters to theory spectra (e.g. from candl.interface.get_CAMB_pars_to_theory_specs_func)
- Grid of candidate band power bin widths

## Outputs

- Fisher matrix at the fiducial point for each scanned configuration
- Parameter covariance matrix (inverse Fisher) and per-parameter 1-sigma errors keyed by par_order
- Table of parameter errors versus bin width with the optimal bin width (e.g. bin_width_scan_results.csv)

## How to apply

Initialise the candl likelihood (candl.Like) from the mock data set .yaml file, rebinning the band powers via the window functions for each candidate bin width on the scan grid. Fix the fiducial cosmological parameter point (e.g. H0=67.37, ombh2=0.02233, omch2=0.1198, logA=3.043, ns=0.9652, tau=0.054, yp=1.0). At each setting, call candl.tools.get_fisher_matrix(pars_to_theory_specs, candl_like, fid_pars, par_order=None, return_par_order=True), which exploits JAX automatic differentiation to obtain exact Hessians of the likelihood rather than finite-difference derivatives. Invert each Fisher matrix to get the parameter covariance, extract 1-sigma errors as the square roots of its diagonal, and use the returned par_order for consistent parameter labelling. Tabulate errors against the scanned setting (e.g. bin width), identify the setting minimising the errors, and validate that derivatives are exact JAX derivatives and that the optimum is a scanned grid point.

## Related tools

- **candl** (Differentiable CMB likelihood providing candl.Like initialisation from .yaml files and candl.tools.get_fisher_matrix for Fisher forecasting) — https://github.com/Lbalkenhol/candl
- **JAX** (Automatic differentiation backend supplying exact gradients and Hessians of the likelihood, avoiding finite differences) — https://github.com/google/jax
- **Python** (Implementation language for the bin-width scan loop and error tabulation)
- **CAMB** (Theory provider via candl.interface.get_CAMB_pars_to_theory_specs_func to map parameters to CMB spectra) — https://github.com/cmbant/CAMB

## Examples

```
par_cov, par_order = candl.tools.get_fisher_matrix(pars_to_theory_specs, candl_like, fid_pars, par_order=None, return_par_order=True)
```

## Evaluation signals

- Fisher matrices are obtained from exact JAX derivatives (Hessians), not finite-difference approximations
- The reported optimal bin width is the scanned grid point at which the minimum parameter error is attained
- 1-sigma errors equal the square roots of the diagonal of the inverted Fisher (parameter covariance) matrix
- Parameter labels match the par_order returned by candl.tools.get_fisher_matrix, with return_par_order=True
- The scan output table (e.g. bin_width_scan_results.csv) covers every candidate bin width on the grid and includes the optimal bin width

## Limitations

- Fisher forecasts are Gaussian approximations at the fiducial point and may not capture non-Gaussian posteriors
- On some architectures 64-bit JAX computation is unavailable, causing up to 1e-4 differences in log-likelihood values (noted in the clipy README)
- Only scanned grid points can be reported as optimal; the optimum is the grid point minimising errors, not a continuous optimum

## Evidence

- [abstract] Fisher via exact JAX derivatives: "This facilitates, for example, robust Fisher matrices without finite-difference methods."
- [abstract] bin-width optimisation example: "These also include optimising the band power bin width to minimise parameter errors of a realistic mock data set."
- [methods] get_fisher_matrix API: "par_cov, par_order = candl.tools.get_fisher_matrix(pars_to_theory_specs, candl_like, fid_pars"
- [intro] JAX Hessians: "JAX-compatibility, allowing for fast and easy computation of gradients and Hessians of the likelihoods."
- [methods] likelihood initialisation: "It simply suffices to point to the ``.yaml`` file of a likelihood to initialise it."
