---
name: band-power-bin-width-optimisation
description: Use when designing or tuning the binning of CMB power spectrum band powers
  (e.g. SPT-3G TT/TE/EE mock data) and you need the bin width that minimises forecast
  parameter errors via Fisher matrices.
when_to_use_negative:
- The goal is parameter inference from real observed data rather than forecast optimisation
  of binning on a mock data set
- Derivatives must come from finite differences or the likelihood is not JAX-differentiable,
  since this skill relies on exact Hessians
- The analysis targets data sets not supported by candl (e.g. non-CMB spectra outside
  TT, TE, EE, BB, phi-phi, kappa-kappa)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3299
- http://edamontology.org/topic_3315
tools:
- name: candl
  role: Differentiable likelihood; supplies candl.Like initialised from .yaml and
    candl.tools.get_fisher_matrix for exact Fisher matrices at each bin width
  repo: https://github.com/Lbalkenhol/candl
- name: JAX
  role: Provides automatic differentiation (gradients and Hessians) of the likelihood,
    avoiding finite-difference Fisher matrices
- name: CAMB
  role: Theory code behind pars_to_theory_specs mapping likelihood parameters to CMB
    spectra for the Fisher forecast
  repo: https://github.com/cmbant/CAMB
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202449432
    title: 'candl: cosmic microwave background analysis with a differentiable likelihood'
  source_content_hash: sha256:b585a56a82a38c716e84ca28ef2f0c60c52a80c8871fc28083ab38fb26d62b3c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lbalkenhol/candl
  tools:
  - CAMB
  - JAX
  - Python
  - candl
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3299
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# band-power-bin-width-optimisation

## Summary

Scans candidate band power bin widths for a mock CMB data set, recomputing candl's exact JAX-differentiated Fisher matrix at each width to identify the binning that minimises parameter 1-sigma errors.

## When to use

Use when designing or tuning the binning of CMB power spectrum band powers (e.g. SPT-3G TT/TE/EE mock data) and you need the bin width that minimises forecast parameter errors via Fisher matrices.

## When NOT to use

- The goal is parameter inference from real observed data rather than forecast optimisation of binning on a mock data set
- Derivatives must come from finite differences or the likelihood is not JAX-differentiable, since this skill relies on exact Hessians
- The analysis targets data sets not supported by candl (e.g. non-CMB spectra outside TT, TE, EE, BB, phi-phi, kappa-kappa)

## Inputs

- Mock CMB band power data set likelihood defined in a .yaml file loadable as candl.Like
- Window functions for rebinning model spectra at each candidate bin width
- Fiducial cosmological parameter point (e.g. H0=67.37, ombh2=0.02233, omch2=0.1198, logA=3.043, ns=0.9652, tau=0.054, yp=1.0)
- pars_to_theory_specs function mapping likelihood parameters to theory spectra (e.g. from candl.interface.get_CAMB_pars_to_theory_specs_func)

## Outputs

- Per-bin-width Fisher matrices and corresponding parameter covariance matrices
- Table of 1-sigma parameter errors versus bin width, written to bin_width_scan_results.csv, including the optimal bin width
- Parameter ordering (par_order) for consistent error labelling

## How to apply

Initialise the candl likelihood (candl.Like) from the mock data set .yaml file and rebin the band powers through the window functions for each candidate bin width on the scan grid. Fix the fiducial cosmological point (e.g. H0=67.37, ombh2=0.02233, omch2=0.1198, logA=3.043, ns=0.9652, tau=0.054, yp=1.0) and, at each bin width, compute the Fisher matrix with candl.tools.get_fisher_matrix(pars_to_theory_specs, candl_like, fid_pars, par_order=None, return_par_order=True), which uses JAX automatic differentiation to obtain exact Hessians rather than finite differences. Invert each Fisher matrix to get the parameter covariance, extract 1-sigma errors as square roots of the diagonal, and keep the returned par_order for consistent labelling. Tabulate errors against bin width, select the grid point attaining the minimum, and write the table (including the optimal bin width) to bin_width_scan_results.csv.

## Related tools

- **candl** (Differentiable likelihood; supplies candl.Like initialised from .yaml and candl.tools.get_fisher_matrix for exact Fisher matrices at each bin width) — https://github.com/Lbalkenhol/candl
- **JAX** (Provides automatic differentiation (gradients and Hessians) of the likelihood, avoiding finite-difference Fisher matrices)
- **CAMB** (Theory code behind pars_to_theory_specs mapping likelihood parameters to CMB spectra for the Fisher forecast) — https://github.com/cmbant/CAMB

## Examples

```
import candl; candl_like = candl.Like('mock_data.yaml'); par_cov, par_order = candl.tools.get_fisher_matrix(pars_to_theory_specs, candl_like, fid_pars, par_order=None, return_par_order=True)
```

## Evaluation signals

- Fisher matrices are obtained from exact JAX derivatives, not finite-difference approximations
- The reported optimal bin width is the scanned grid point at which the minimum parameter error is attained
- 1-sigma errors equal the square roots of the diagonal of the inverted Fisher matrix, labelled consistently via the returned par_order
- Each Fisher matrix is computed at the same fixed fiducial point (H0=67.37, ombh2=0.02233, omch2=0.1198, logA=3.043, ns=0.9652, tau=0.054, yp=1.0)
- bin_width_scan_results.csv contains error-versus-bin-width rows for the full scan grid plus the identified optimal bin width

## Limitations

- Optimal bin width is only valid for the fiducial cosmology and mock data set used; it is not a globally optimal binning
- The optimum is chosen over the discrete scanned grid of candidate bin widths, not a continuous optimisation
- Results depend on JAX 64-bit precision being available; JAX support issues on some architectures can perturb likelihood computations (clipy README notes up to 1e-4 differences without 64-bit support)

## Evidence

- [abstract] bin width optimisation via Fisher forecasts: "These also include optimising the band power bin width to minimise parameter errors of a realistic mock data set."
- [abstract] exact Fisher matrices without finite differences: "This facilitates, for example, robust Fisher matrices without finite-difference methods."
- [methods] Fisher matrix utility call signature: "par_cov, par_order = candl.tools.get_fisher_matrix(pars_to_theory_specs, candl_like, fid_pars"
- [methods] likelihood initialisation from yaml: "It simply suffices to point to the ``.yaml`` file of a likelihood to initialise it."
- [methods] binning model spectra: "#. Bin the model spectra using the window functions."
- [readme] JAX gradient and Hessian support: [evidence span withheld — anchor status: external_doc]
