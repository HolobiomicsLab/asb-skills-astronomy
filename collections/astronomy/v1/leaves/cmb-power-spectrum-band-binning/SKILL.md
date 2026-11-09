---
name: cmb-power-spectrum-band-binning
description: Use when designing or re-binning a CMB power spectrum likelihood (e.g.
  SPT-3G TT/TE/EE or a mock data set in .yaml form) and you need to choose the band
  power bin width that minimises parameter sigma errors via Fisher forecasts.
when_to_use_negative:
- When the goal is posterior sampling or best-fit optimisation rather than forecast
  errors — Fisher forecasts here assume a Gaussian likelihood around a fixed fiducial
  cosmology
- When the likelihood is not JAX-differentiable (e.g. non-candl likelihoods), since
  the method relies on exact Hessians and would otherwise require finite differences
- When bin width is already fixed by the published data release and cannot be re-tuned
  (rebinning requires access to window functions and un-binned model spectra)
tools:
- name: candl
  role: Differentiable CMB likelihood used to bin model spectra with window functions
    and compute Fisher matrices via candl.tools.get_fisher_matrix
  repo: github.com/Lbalkenhol/candl
- name: JAX
  role: Automatic differentiation backend providing exact gradients and Hessians of
    the likelihood, avoiding finite-difference methods
- name: Python
  role: Implementation language for the bin width scan loop, covariance inversion,
    and CSV tabulation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1051/0004-6361/202449432
    title: 'candl: cosmic microwave background analysis with a differentiable likelihood'
  source_content_hash: sha256:20b674d643298f5af11eb427a23ac08425ccf1a4aedcfd5fd657d9d818841cd2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - JAX
  - Python
  - candl
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cmb-power-spectrum-band-binning

## Summary

A Fisher-forecast-based optimisation that scans candidate CMB band power bin widths and selects the one minimising cosmological parameter errors, exploiting candl's JAX-differentiable likelihood for exact Hessians instead of finite differences.

## When to use

Use when designing or re-binning a CMB power spectrum likelihood (e.g. SPT-3G TT/TE/EE or a mock data set in .yaml form) and you need to choose the band power bin width that minimises parameter sigma errors via Fisher forecasts.

## When NOT to use

- When the goal is posterior sampling or best-fit optimisation rather than forecast errors — Fisher forecasts here assume a Gaussian likelihood around a fixed fiducial cosmology
- When the likelihood is not JAX-differentiable (e.g. non-candl likelihoods), since the method relies on exact Hessians and would otherwise require finite differences
- When bin width is already fixed by the published data release and cannot be re-tuned (rebinning requires access to window functions and un-binned model spectra)

## Inputs

- candl likelihood .yaml file for a realistic mock data set (e.g. an SPT-3G-style TT/TE/EE mock)
- pars_to_theory_specs function mapping cosmological parameters to theory spectra (e.g. built via candl.interface.get_CAMB_pars_to_theory_specs_func)
- Fiducial parameter dictionary (H0=67.37, ombh2=0.02233, omch2=0.1198, logA=3.043, ns=0.9652, tau=0.054, yp=1.0)
- Grid of candidate band power bin widths

## Outputs

- Fisher matrix and parameter covariance at the fiducial point for each candidate bin width
- 1-sigma parameter errors (square roots of covariance diagonal) per bin width, labelled by the returned par_order
- bin_width_scan_results.csv tabulating parameter errors versus bin width, including the optimal bin width

## How to apply

Initialise the candl likelihood (candl.Like) from the mock data set .yaml file and fix a fiducial parameter point (e.g. H0=67.37, ombh2=0.02233, omch2=0.1198, logA=3.043, ns=0.9652, tau=0.054, yp=1.0). For each candidate bin width on the scan grid, rebin the band powers via the window functions, then compute the Fisher matrix at the fiducial point with candl.tools.get_fisher_matrix(pars_to_theory_specs, candl_like, fid_pars, par_order=None, return_par_order=True), which uses JAX automatic differentiation for exact Hessians. Invert each Fisher matrix to get the parameter covariance, take square roots of the diagonal as the 1-sigma errors, and keep the returned par_order for consistent labelling. Tabulate errors against bin width, identify the grid point attaining the minimum, and write the table including the optimal bin width to a CSV such as bin_width_scan_results.csv.

## Related tools

- **candl** (Differentiable CMB likelihood used to bin model spectra with window functions and compute Fisher matrices via candl.tools.get_fisher_matrix) — github.com/Lbalkenhol/candl
- **JAX** (Automatic differentiation backend providing exact gradients and Hessians of the likelihood, avoiding finite-difference methods)
- **Python** (Implementation language for the bin width scan loop, covariance inversion, and CSV tabulation)

## Examples

```
par_cov, par_order = candl.tools.get_fisher_matrix(pars_to_theory_specs, candl_like, fid_pars, par_order=None, return_par_order=True)
```

## Evaluation signals

- Fisher matrices are obtained from exact JAX derivatives (Hessians), not finite-difference approximations
- The reported optimal bin width equals the scanned grid point at which the minimum parameter error is attained when errors are tabulated against bin width
- Parameter errors match the square roots of the diagonal of the inverted Fisher matrix (parameter covariance)
- Parameter labels are consistent across bin widths via the par_order returned with return_par_order=True
- Each scan point uses the same fixed fiducial cosmology (e.g. H0=67.37, ombh2=0.02233, omch2=0.1198) so comparisons across bin widths are apples-to-apples

## Limitations

- Fisher forecasts are local to the chosen fiducial parameter point, so the optimal bin width is only optimal for that fiducial cosmology and parameter set
- Only the scanned grid of bin widths is explored; the optimum is the best grid point, not a globally optimised continuous value
- JAX on some architectures lacks 64-bit precision, which can shift log-likelihood values at the ~1e-4 level (per the clipy README's JAX notes), potentially affecting sensitive error comparisons

## Evidence

- [abstract] Optimising band power bin width via Fisher forecasts: "These also include optimising the band power bin width to minimise parameter errors of a realistic mock data set."
- [abstract] Exact derivatives enable robust Fisher matrices: "This facilitates, for example, robust Fisher matrices without finite-difference methods."
- [intro] JAX-compatibility for gradients and Hessians: "JAX-compatibility, allowing for fast and easy computation of gradients and Hessians of the likelihoods."
- [methods] Fisher matrix API with par_order: "par_cov, par_order = candl.tools.get_fisher_matrix(pars_to_theory_specs, candl_like, fid_pars"
- [methods] Binning model spectra with window functions: "#. Bin the model spectra using the window functions."
- [methods] Initialising a likelihood from a .yaml file: "It simply suffices to point to the ``.yaml`` file of a likelihood to initialise it."
