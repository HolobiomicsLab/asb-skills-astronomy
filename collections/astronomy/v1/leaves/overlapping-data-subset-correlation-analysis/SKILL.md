---
name: overlapping-data-subset-correlation-analysis
description: Use when you need to quantify how strongly parameter constraints from
  correlated or partially overlapping subsets of a CMB power-spectrum data set (e.g.
  SPT-3G 2018 TT/TE/EE) agree, via Fisher-matrix covariances rather than full MCMC
  runs.
when_to_use_negative:
- The input is a single, non-overlapping data set with no subset structure — there
  is no second covariance to correlate against.
- Full posterior (non-Gaussian) constraint comparison is required; Fisher matrices
  assume a Gaussian likelihood in parameter space.
- The likelihood or data set is not one of the candl-supported power-spectrum releases
  (the workflow depends on candl_data and JAX-differentiable likelihoods).
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: candl
  role: Differentiable CMB likelihood framework; provides Like initialisation, data_selection
    keyword, and candl.tools.get_fisher_matrix used to compute per-subset Fisher matrices
  repo: https://github.com/Lbalkenhol/candl
- name: JAX
  role: Powers automatic differentiation of the likelihood so Fisher matrices are
    computed without finite differences
- name: candl_data
  role: Supplies the SPT3G_2018_TTTEEE data (band powers, covariance, window functions)
    being subsetted
  repo: https://github.com/Lbalkenhol/candl_data
- name: Python
  role: Implementation language for the subset-construction, Fisher computation, and
    correlation-coefficient output script
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1051/0004-6361/202449432
    title: 'candl: cosmic microwave background analysis with a differentiable likelihood'
  source_content_hash: sha256:fb08e120f51e90ca1a672bbc5c4315d25ba0790c6e04ca25917ba2e0dde4fb73
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lbalkenhol/candl
  tools:
  - JAX
  - Python
  - candl
  - candl_data
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# overlapping-data-subset-correlation-analysis

## Summary

Quantifies the cross-correlation of cosmological parameter constraints between correlated and partially overlapping data subsets (here, SPT-3G 2018 TT/TE/EE band powers) by computing per-subset Fisher matrices with JAX automatic differentiation via candl and correlating the resulting parameter covariances.

## When to use

Use when you need to quantify how strongly parameter constraints from correlated or partially overlapping subsets of a CMB power-spectrum data set (e.g. SPT-3G 2018 TT/TE/EE) agree, via Fisher-matrix covariances rather than full MCMC runs.

## When NOT to use

- The input is a single, non-overlapping data set with no subset structure — there is no second covariance to correlate against.
- Full posterior (non-Gaussian) constraint comparison is required; Fisher matrices assume a Gaussian likelihood in parameter space.
- The likelihood or data set is not one of the candl-supported power-spectrum releases (the workflow depends on candl_data and JAX-differentiable likelihoods).

## Inputs

- candl likelihood initialised from candl_data.SPT3G_2018_TTTEEE (or its .yaml info file)
- Fiducial cosmological parameter point (Planck 18 best fit: H0=67.37, ombh2=0.02233, omch2=0.1198, logA=3.043, ns=0.9652, tau=0.054, yp=1.0)
- data_selection specifications defining multipole cuts / band-power sub-selections for each correlated or partially overlapping subset

## Outputs

- Per-subset Fisher matrices and parameter covariances (par_cov, with par_order)
- Parameter-by-subset correlation coefficients written to subset_parameter_correlation.csv

## How to apply

Install candl (pip install candl-like) together with the candl_data repository, and initialise the likelihood with candl.Like pointing to candl_data.SPT3G_2018_TTTEEE or its .yaml info file. Define the fiducial point at the Planck 18 best fit (H0=67.37, ombh2=0.02233, omch2=0.1198, logA=3.043, ns=0.9652, tau=0.054, yp=1.0). Build correlated and partially overlapping variants of the data set using the data_selection keyword to apply multipole cuts / sub-selections of the band powers. For each subset, compute the Fisher matrix with candl.tools.get_fisher_matrix (return_par_order=True), exploiting JAX automatic differentiation so no finite-difference stepping is needed, then invert each Fisher matrix to get the parameter covariance and compute parameter-by-subset cross-correlation coefficients. Because the Fisher approach is analytic, the whole comparison is cheap compared to sampling-based alternatives, which is the rationale for using candl's differentiable likelihood here.

## Related tools

- **candl** (Differentiable CMB likelihood framework; provides Like initialisation, data_selection keyword, and candl.tools.get_fisher_matrix used to compute per-subset Fisher matrices) — https://github.com/Lbalkenhol/candl
- **JAX** (Powers automatic differentiation of the likelihood so Fisher matrices are computed without finite differences)
- **candl_data** (Supplies the SPT3G_2018_TTTEEE data (band powers, covariance, window functions) being subsetted) — https://github.com/Lbalkenhol/candl_data
- **Python** (Implementation language for the subset-construction, Fisher computation, and correlation-coefficient output script)

## Examples

```
import candl; like = candl.Like('SPT3G_2018_TTTEEE'); par_cov, par_order = candl.tools.get_fisher_matrix(pars_to_theory_specs, like, fid_pars, return_par_order=True)
```

## Evaluation signals

- All reported parameter correlation coefficients lie within [-1, 1].
- A subset correlated with itself yields a correlation coefficient of exactly 1.
- Inverting each Fisher matrix must return a positive-definite parameter covariance (par_cov).
- Correlation coefficients are produced for every parameter in par_order and written to subset_parameter_correlation.csv.
- Results should be consistent with the paper's reported correlation calculation for SPT-3G 2018 TT/TE/EE subsets.

## Limitations

- candl internal priors are ignored by default in recent versions (breaking change in default behaviour); check docs for how to keep them.
- Some architectures do not support 64-bit computation in JAX, which can cause up to ~1e-4 differences in log-likelihood computations (noted for clipy's Planck likelihoods).
- Fisher-matrix correlations assume local Gaussianity around the fiducial Planck 18 best-fit point; they are not a substitute for full posterior overlap analysis.

## Evidence

- [abstract] Fisher matrices without finite differences: "This facilitates, for example, robust Fisher matrices without finite-difference methods."
- [abstract] Correlation of parameter constraints from overlapping subsets: "we calculate the correlation of parameter constraints from correlated and partially overlapping subsets of the SPT-3G 2018 TT/TE/EE data release"
- [methods] data_selection keyword for sub-setting: "data_selection: ... # Select a subset of the data set"
- [methods] get_fisher_matrix API: "par_cov, par_order = candl.tools.get_fisher_matrix(pars_to_theory_specs, candl_like, fid_pars"
- [intro] JAX compatibility rationale: "JAX-compatibility, allowing for fast and easy computation of gradients and Hessians of the likelihoods."
